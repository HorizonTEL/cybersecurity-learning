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

##### 二、nmap
```
活跃主机识别:
nmap -sT tcp扫描，普通扫描
nmap -sS 秘密扫描，没有建立tcp连接，所以日志没有痕迹
nmap -sL list扫描
nmap -sn 使用ping，只检测主机是否存活
namp -Pn 跳过主机存活扫描过程
nmap -sP [网段]

nping -tcp -p 445 -data [16进制数据] [目标ip] 对目标的445端口，以tcp的方式发送一串数据
(16进制数据可以有意义，也可以无意义。可以通过上述方法进行模拟常见的网络攻击，可以测试对方的防御效果)

【注】zenamp是nmap的图形化界面

nmap -sV [目标ip] 服务器指纹识别
amap -bq [目标ip] [端口号] 

标准扫描主机开放的端口 : nmap -p -T4 [目标ip]
汇报端口详细信息	  : nmap -T4 -A -v [目标ip] 服务枚举
```
