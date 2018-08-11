# 20180811_ctf

## vim编辑漏洞

 一.vim备份文件

产生原因：默认情况下使用vim编程，在修改文件后系统会自动生成一个带~
备份文件，某些情况下可以对其进行下载查看，从而造成源码泄露。eg：index.php是首页，它的备份文件为index.php~

二.vim临时文件

vim中的swp即swap文件，在编辑文件时产生，属于隐藏文件。eg：源文件名为test，则它的临时文件是.test.swap。如果文件正常退出，该文件自动删除。

## LFI漏洞（Local File Include）

### 漏洞简介

LFI是能打开包括`本地`文件的漏洞；区别RFI，远程包含文件漏洞；

意义：文件包含漏洞是"代码注入"的一种，包含即执行。危害：

1. PHP包含漏洞结合上传漏洞；
2. PHP包含读文件；
3. PHP包含写文件；
4. PHP包含日志文件；
5. PHP截断包含；
6. PHP内置伪协议利用。

PHP中文件包含函数有以下四种：

1. require()
2. require_once()
3. include()
4. include_once()

include()和require()的主要区别是，include()在包含过程中，如果出现错误，会抛出一个警告，程序为抛出warning但程序继续执行，而require不会，所有有@include()。

最简单的漏洞代码：`<?php include(_$GET[file]);?>`

当使用这4个函数包含一个新的文件时，该文件将作为PHP代码执行，PHP的内核并不会在意被包含的文件是什么类型。即你可以上传一个含shell的txt或jpg文件，包含它会被当作PHP代码执行（图马）。

## 与ctf关系（协议基础）

1.`php：//`伪协议>>访问 访问各个输入/输出流

- php://filter
  
1. 解释：php://filter 是一种元封装器，设计用于"数据流打开"时的筛选功能，对本地磁盘文件进行读写。可以将文件在执行代码钱换个方式读取出来

2. 用法:file=php://filter/convert.base64-encode/resource=xxx.php；

3. ?file=php://filter/read=convert.base64-encode/resource=xxx.php 一样

- php://input

1. 解释：上面的filter文件能读文件也能写文件，于是可以通过input将数据Post过去，php://input使用率接收post数据的

2. 用法：?file=php://input 数据利用POST传过去

3. 注意：如果php.ini里的allow_url_include=On（PHP < 5.30）,就可以造成任意代码执行，在这可以理解成远程文件包含漏洞（RFI），即POST过去一句话，如，即可执行；

4. 例子：碰到file_get_contents()`屏蔽了特定词` 就要想到用php://input绕过，因为php伪协议也是可以利用http协议的，即可以使用POST方式传数据；

2.`data://`数据流>>数据流封装器，利用了流的概念，将原本的include文件流重定向到了用户可控制的输入流中，执行文件的包含方法包含了你的输入流，通过输入payload来实现目的

- data://text/plain

1. 用法：?file=data://text/plain;base64,base64编码的payload
2. 注意：

        - `<?php phpinfo();`,这类执行代码最后没有?>闭合；

        - 如果php.ini里的allow\_url_include=On（PHP < 5.30）,就可以造成任意代码执行，同理在这就可以理解成远程文件包含漏洞（RFI）；
     1. 例子：
        - 和php伪协议的input类似，碰到file\_get_contents()来用；
        - 练习题目源码文件见data文件夹；

3. `phar://`伪协议 >> 数据流包装器，自 PHP 5.3.0 起开始有效，正好契合上面两个伪协议的利用条件。说通俗点就是php解压缩包的一个函数，解压的压缩包与后缀无关。
   - phar://
     1. 用法：?file=phar://压缩包/内部文件
     2. 注意：

        - PHP版本需大于等于 5.3，这就说明上述协议已经挂掉了，但又出来了phar协议前赴后继；
        - 压缩包一般是phar后缀，需要代码来生成，但是zip后缀也可以；
        - 压缩包需要是zip协议压缩，rar不行，tar等格式待测；
        - 利用url的压缩包后缀可以是任意后缀；
     3. 例子：
        - 本地：phar1文件（SWPU2016，限制上传类型）
        - 本地：phar2文件（限制上传类型，上传重命名）

4. 上述说的php.ini文件的限制如下：
   - allow\_url_fopen = On `默认打开` ，允许URLs作为像files一样作为打开的对象；
   - allow\_url_include = On `默认关闭` ，允许include/require函数像打开文件一样打开URLs；