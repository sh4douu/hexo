---
title: WEB漏洞靶场Pikachu Writeup Chapter 3：CSRF
date: 2019-03-25 13:09:32
tags: 
	- 靶场
	- WEB安全
	- XSS
categories: Pikachu
---

### CSRF之GET提交数据

打开页面，是一个登录页面，我们使用lucy/123456进入用户后台。

<!-- more -->

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325131233.png)

进入用户后台后，点击修改个人信息，修改信息后点击提交并使用Burp进行抓包。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325131618.png)

观察Burp抓到的数据包可以发现，数据包是以GET方式提交的，参数与表单的输入框也是一一对应的，没有任何CSRF防御。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325131837.png)

那么我们只需要修改 `http://192.168.100.111/pikachu/vul/csrf/csrfget/csrf_get_edit.php?sex=female&phonenum=12345678922&add=US&email=lucy%40pikachu.com&submit=submit`这个链接的参数值部分，然后发给想要攻击的用户，若用户本地存在身份认证信息（Cookie），那么他的个人信息就会被修改为我们链接里面的信息。

**漏洞分析**

通过源码可以得知，服务器接收到用户提交的修改个人信息的请求时，只判断是否登录以及是否有信息为空，若不为空就直接更新个人信息数据表，没有进行任何CSRF验证。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325132842.png)

### CSRF之POST提交数据

同上一节一样，登录然后修改信息抓包，不同的是这次我们抓包发现数据是以POST方式提交的。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325133302.png)

然后右键数据包，按下图方式选择生成CSRF PoC。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325133440.png)

根据下图修改信息生成PoC：

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325133950.png)

将拷贝的PoC存储为evil.html，并且放于我们的服务器上。然后将该PoC文件的URL地址发给想要攻击的用户，用户点击访问该PoC文件，PoC文件的表单数据就会被提交到服务器，导致用户信息被修改。

**漏洞分析**

与上一节基本没什么区别，区别只是提交数据的方式不同。

### Token防止CSRF

接下来看Token是如何防止CSRF的。

点击修改个人信息按钮后，服务器会返回一个个人信息的表单页面。表单会嵌入一个隐藏的`<input />`标签，标签的`value`属性值为服务器端生成并存储在Session中的token，表单数据被提交时，token也会被提交到服务器。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325140018.png)

当填好个人信息的表单，点击提交后，服务器不仅检查信息是否有空值，还会将表单提交过来的token值也Session中的token值作对比，对比不通过不会执行个人信息修改操作。

![](WEB漏洞靶场pikachu-CSRF\QQ截图20190325140514.png)

Token防止CSRF的本质是让攻击者无法完整预测数据包的参数部分。