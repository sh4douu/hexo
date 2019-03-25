---
title: NetBIOS、SMB浅析
date: 2019-03-23 11:54:48
tags:
	- SMB
	- NetBIOS
categories: Misc
---

## 前面的话

一直以来对NetBIOS和SMB很模糊，也分不清137、138、139、445端口的作用。

借此有时间想进行梳理并记录以便自己理解。

<!-- more -->

## 正文

### NetBIOS

> NetBIOS，为网络基本输入输出系统（英语：Network Basic Input/Output System）的缩写，它提供了OSI模型中的会话层服务，让在不同计算机上运行的不同程序，可以在局域网中，互相连线，以及分享数据。严格来说，NetBIOS不是一种网络协议，而是应用程序接口（API）。  ——摘自Wikipedia

NetBIOS提供了三种软件服务：

> - 名称解析服务（NetBIOS Name Service）：UDP 137
> - 数据报服务（NetBIOS Datagram Service）：UDP 138
> - 会话服务（NetBIOS Session Service）：TCP 139

![](NetBIOS、SMB浅析\20181112194501454.jpg)

**0x00 名称解析服务（NBNS）**

运行在137（UDP）端口，提供计算机的名字到IP地址的查询服务，类似于TCP/IP协议中的DNS。

计算机名称到IP地址的管理方式：

- 第一种：位于同一工作组中的电脑之间利用广播功能进行计算机名管理。

> ​	电脑在启动或者连接网络时，会向同一工作组中的所有计算机质询有没有和自己相同的NetBIOS名称。

- 另一种：利用WINS（Windows因特网名称服务）管理NetBIOS名称。

> WINS服务器用于记录计算机NetBIOS名称和IP地址的对应关系，供局域网计算机查询。WINS客户端在系统起动时或连接网络时会将自己的NetBIOS名称与IP地址发送给WINS服务器。

向目标主机的137端口发送一个连接请求，就能获得目标主机的名称、MAC地址。

#### NBTSTAT

nbtstat是Windows下与NetBIOS相关的DOS命令。

```
NBTSTAT [ [-a RemoteName] [-A IP address] [-c] [-n] [-r] [-R] [-RR] [-s] [-S] [interval] ]

  -a   (适配器状态)     列出指定名称的远程机器的名称表（可跟名称和IP）
  -A   (适配器状态)     列出指定 IP 地址的远程机器的名称表。（只能IP）
  -c   (缓存)          列出远程[计算机]名称及其 IP 地址的 NBT 缓存
  -n   (名称)          列出本地 NetBIOS 名称。
  -r   (已解析)        列出通过广播和经由 WINS 解析的名称
  -R   (重新加载)       清除和重新加载远程缓存名称表
  -S   (会话)          列出具有目标 IP 地址的会话表
  -s   (会话)          列出将目标 IP 地址转换成计算机 NETBIOS 名称的会话表。
  -RR  (释放刷新)      将名称释放包发送到 WINS，然后启动刷新

  RemoteName   远程主机计算机名。
  IP address   用点分隔的十进制表示的 IP 地址。
  interval     重新显示选定的统计、每次显示之间暂停的间隔秒数。
               按 Ctrl+C 停止重新显示统计。
```

环境介绍：

我们的局域网中存在两台主机，分别是：

> 主机名（NetBIOS名称）  <=========>       IP地址
>
> ​    SAKURA_II          <=========>    192.168.1.105 
>
> ​    XESTATION          <=========>    192.168.1.101

以下演示的操作均在IP为192.168.1.105的主机上进行，并使用Wireshark抓包分析。

- **`nbtstat -A 192.168.1.101`** ：查询IP为192.168.1.101主机的计算机的名称列表。

> 实质是向192.168.1.101主机的NetBIOS的名称解析服务（NBNS)监听的UDP 137端口发送查询。

Wireshark抓取请求包：

![](NetBIOS、SMB浅析\QQ截图20190323121259.png)

命令执行的结果：

![](NetBIOS、SMB浅析\QQ截图20190323122802.png)



- **`nbtstat -c`**：查询NetBIOS缓存。
- **`nbtstat -n`**：查询本地NetBIOS名称。

**0x01 数据报服务（NBDS）**

运行在138（UDP）端口，提供NetBIOS浏览功能，即显示连接到网络的计算机设备列表。

> - 每台电脑在启动时或连接网络时通过138端口广播自己的NetBIOS名称，收到NetBIOS广播的计算机会将该计算机追加到浏览列表中；
> - 关闭电脑时，计算机会通过138端口广播，收到NetBIOS广播的计算机会将该计算机从浏览列表中删除；
> - 当计算机需要连接加入到网络的计算机设备列表，会广播一个请求，收到请求的主机会给其发送计算机列表。

向目标主机的138端口发送请求，就能获得目标主机所处的局域网的网络名称以及目标主机的计算机名称，以及是否安装主控控制器，IIS是否正在运行等。

周期性在UDP 138端口广播NetBIOS信息：

![](NetBIOS、SMB浅析\QQ截图20190323125719.png)

**0x02 会话服务（NBSS）**

运行在139（TCP）端口，提供“NetBIOS Session Service”服务，使用SMB协议对外提供共享服务，包括Windows文件和打印机共享以及Unix中的Samba服务。

***********

### SMB协议

SMB（Server Message Block）是由微软开发的应用层网络传输协议，主要功能是使网络上的机器能够共享计算机文件、打印机、串行端口和通讯等资源。

经过Unix服务器厂商重新开发后，它可以用于连接Unix服务器和Windows客户机，执行打印和文件共享等任务。

SMB一开始的设计是在NetBIOS协议上运行的（而NetBIOS本身则运行在NetBEUI、IPX/SPX或TCP/IP协议上），从Windows 2000开始，微软引入SMB Direct Over TCP，重新命名为 CIFS（Common Internet File System），并打算将它与NetBIOS相脱离。

#### CIFS

CIFS是由微软在SMB的基础上，扩展到Internet上的协议。若SMB直接运行在TCP上（而不是NetBIOS上），即CIFS协议，使用TCP端口号445。提供的功能与139端口完全相同。

为保证向后兼容性，Windows 2000以后版本中基于NBT的SMB和CIFS同时并存。

> 当Windows系统（允许NBT）来连接SMB服务器时，同时尝试连接139和445端口：
>
> - 如果445端口有响应，那么发送RST包给139端口断开连接，使用455端口提供SMB服务。
> - 当445端口无响应时，使用139端口提供SMB服务。

**参考资料**

https://www.cnblogs.com/bigrabbit/p/3910094.html

#### Wireshark抓包

在主机192.168.1.105上开启文件共享，不需要密码。

在主机192.168.1.101上访问192.168.1.105主机上的共享：\\192.168.1.105

![](NetBIOS、SMB浅析\QQ截图20190323140801.png)

