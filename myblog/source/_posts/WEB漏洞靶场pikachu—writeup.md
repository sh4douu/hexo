---
title: WEB漏洞靶场Pikachu Writeup Chapter 1：表单暴力破解
date: 2019-03-19 09:03:38
tags: 
	- 靶场
	- WEB安全
	- 暴力破解
categories:
	- Pikachu
---

## 一、前面的话

### 0x00 Whatever

无意间发现了pikachu这个WEB漏洞靶场，发现靶场涉及到的漏洞类型蛮多的，练练手挺合适的。

看到靶场介绍里有一句话：

> “如果你想搞懂一个漏洞，比较好的方法是：你可以自己先制造出这个漏洞（用代码编写），然后再利用它，最后再修复它”。

觉得很是这么个道理，因此在使用该靶场时，会包含如下方面：

> - 漏洞挖掘：找出漏洞或证明漏洞。
> - 漏洞分析：分析源代码。
> - 漏洞利用。
> - 漏洞修复。

<!-- more -->

### 0x01 靶机介绍

Pikachu源码：https://github.com/zhuifengshaonianhanlu/pikachu

Pikachu是一个带有漏洞的Web应用系统，包含了常见的web安全漏洞。

包含：

> - Burt Force(暴力破解漏洞)
> - XSS(跨站脚本漏洞)
> - CSRF(跨站请求伪造)
> - SQL-Inject(SQL注入漏洞)
> - RCE(远程命令/代码执行)
> - Files Inclusion(文件包含漏洞)
> - Unsafe file downloads(不安全的文件下载)
> - Unsafe file uploads(不安全的文件上传)
> - Over Permisson(越权漏洞)
> - ../../../(目录遍历)
> - I can see your ABC(敏感信息泄露)
> - PHP反序列化漏洞
> - XXE(XML External Entity attack)
> - 不安全的URL重定向
> - SSRF(Server-Side Request Forgery)
> - 管理工具
> - More...(找找看?..有彩蛋!)

## 二、暴力破解

### 0x00 简单表单爆破

**漏洞挖掘**

打开页面，是一个登录表单页面。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319090517.png)

输入admin/admin作为用户名/密码，点击Login，并使用burp进行抓包拦截。

在burp右键抓到的数据包，然后`Send to Intruder`。或者直接`Ctrl + I`发送到intuder爆破模块。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319090805.png)

转到`intruder`模块，在`Positions`按钮下，先点击右侧的`Clear $ `清除变量，然后选中用户名和密码位置点击`ADD $`将用户名和密码位置作为变量。`Attack type`就使用`Cluster bomb`模式。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319093205.png)

点击`Positions`按钮旁边的`Payloads`按钮进入设置Payload，由于我们设置了两个变量，所以需要设置两个Payload。

选择`Payload set`为1，`Payload Type`使用默认的`Simple list`，然后点击`Load`导入用户名字典文件。

选择`Payload set`为2，`Payload Type`使用默认的`Simple list`，然后点击`Load`导入密码字典文件。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319093646.png)

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319093717.png)

设置好之后，点击右上角`Start Attack`。

在爆破时，点击`Length`字段进行排序，可发现有不同大小的数据包，点击查看数据包发现爆破成功。密码：admin/123456

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319093939.png)

### 0x01 服务器端验证码绕过

**漏洞挖掘**

打开页面，还是一个登录表单页面，但是增加了验证码。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319094111.png)

接下来尝试绕过验证码。

首先先确定，验证码是服务器生成的，还是客户端JS生成的。

在网页上按下`F12`按钮，定位验证码图片，图片由`<img>`标签加载，可以确定验证码由服务器生成。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319104247.png)

确定了验证码是由服务器生成了之后，再确定服务器是否真的对验证码做了验证。

输入用户名密码及验证码然后点击Login，并使用Burp进行抓包。

右击抓到的数据包，选择`Send to Repeater`发送到重放模块。或者直接`Ctrl + R`。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319094958.png)

点击进入`Repeater`模块，将验证码修改为任意值后，点击`Go`按钮进行重放。发现返回验证码输入错误，证明服务器对验证码做了验证。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319095233.png)

接下来，将验证码设置为正确的值，然后发送数据包，发现验证码验证成功。然后再继续重放该正确验证码的数据包几次，发现即使由于用户名密码不正确，导致页面刷新了验证码，但是并不需要再次输入验证码。即服务器只验证了一次验证码，验证成功的验证码可以一直重复使用。

我们只需要验证正确一次验证码，就可以一直使用该验证码，然后就可以进行爆破了。爆破的流程同上一节，不再赘述。

**漏洞分析：**

接下来，让我们来分析一下源码，看漏洞是如何产生的（只截取部分关键代码）：

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319101955.png)

通过源码分析，我们可以得知，我们提交的验证码会被与存储在服务器session中的验证码做验证，但是由于在验证完成后，没有及时的销毁session，导致存储在session中的验证码在session生存周期内可以一直被重复使用。

**漏洞修复**：

在验证完成之后，销毁验证码session：`unset($_SESSION['vcode']);`

另，

通过分析验证码生成的源码，还发现了一个问题，生成的验证码会被以明文方式作为Cookie返回给客户端。这会导致直接可以进行爆破，爆破方法参照下面0x03节的内容。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319103538.png)

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319103611.png)

### 0x02 客户端验证码绕过

**漏洞挖掘**

打开页面，是一个包含验证码的表单登录页面。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319103858.png)

还是先确定验证码是由JS生成还是由服务器生成。

继续在页面`F12`,然后定位找到元素，可以确定是由客户端JS的`createCode()`函数生成。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319104637.png)

输入用户名密码及验证码，点击Login并使用Burp抓包。将验证码删除并进行重放，发现没有影响。可以确定服务器并未对验证码进行验证。接下来直接进行爆破即可，爆破流程参考前面。

**漏洞分析**

前端JS生成验证码且前端JS进行验证码的校验，对使用Burp抓包爆破而言形同虚设。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319110240.png)

### 0x03 Token防爆破

首先，先给出结论，表单token的作用是防止重复提交和CSRF。但不要认为有防止重复提交在防爆破上就有作用。

因为表单Token是作为一个隐藏的`<input />`标签的值输出在返回给用户的页面的，是可以直接从页面读取获取的，所以是防不了爆破的。如果还有疑问，继续往下看使用Burp进行爆破。

还是继续登录并抓包的操作，可以发现POST提交的数据中存在Token，第一次放行数据包，返回用户名密码不存在。接着再连续重放该数据包，发现只是刷新的页面，返回消息没有再报用户名密码不存在。因此可以猜测到，服务器对Token做了验证，由于Token不正确所以没有进行用户名密码验证的逻辑，所以仅刷新了页面。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319114454.png)

接下来使用Burp进行爆破，先将数据包发送到Intruder模块。

将用户名、密码、Token作为变量，`Attack Type`使用`Pitch fork`模式。（注意，只能使用该模式）

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319115718.png)

转到`Options`按钮下，找到`Grep-Extract`，勾选`Extract the following items from response`，然后点击`ADD`。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319125134.png)

点击`ADD`后会弹出一个框，按下图步骤方式配置，配置完成点击`OK`。作用就是可以从响应消息（Response）中提取一段数据作为下一次请求信息（Request）的Payload。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319125941.png)

接着转到`Payloads`按钮下设置Payload

选择`Payload set`为1，`Payload Type`使用默认的`Simple list`，然后点击Load导入用户名字典文件。

选择`Payload set`为2，`Payload Type`使用默认的`Simple list`，然后点击Load导入密码字典文件。

选择`Payload set`为3，`Payload Type`选择`Recursive grep`，然后输入第一个请求要使用的Payload(可以不设)。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319130503.png)

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319130947.png)

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319131320.png)

设置完成后，点击`Start attack`，发现报错。这是因为不能使用多线程爆破，因为每次爆破其中一个Payload需要从前一个响应消息中提取。因此再次转到`Options`按钮下，找到`Request Engine`，修改线程数为单线程。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319131802.png)

再次点击`Start attack`，进行爆破即可。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319132139.png)

点击`Length`按钮进行排序，发现一个长度与其他不一致的数据包，点击数据包查看，爆破成功。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319132951.png)

**漏洞分析**

查看源码发现服务器端在进行用户名密码验证的逻辑之前，先验证了token。印证了我们之前的猜想。

![](WEB漏洞靶场pikachu—writeup\QQ截图20190319133447.png)

但是使用Token进行防爆破的漏洞不在于是否对Token进行了验证，而在于Token会返回在页面中被读取。

## 三、总结

> 防暴力破解应该理解成防止自动化脚本快速请求以达到破解的目的。

**表单暴力破解防范**：

>1. 使用安全的验证码进行验证（滑动验证码...）。
>2. 在某段单位时间内，用户登录失败达到一定次数时，锁定用户或限制短时间内不允许登录。
>3. 对IP进行单位时间内登录请求数的限制。
>4. 验证码应该以图片形式而不是以字符串形式出现在HTML或Cookie中。

**参考链接：**

https://www.cnblogs.com/KbCat/p/9317545.html