---
title: Windows认证之Kerberos
date: 2019-04-03 11:58:35
tags: 
	- Kerberos
categories: 内网渗透
---

### 0x00 Kerberos

![](02-Windows认证之Kerberos\cerberus.jpg)

Kerberos起源于希腊神话，是一只守护着冥界长着3个头颅的神犬，在Kerberos认证中，Kerberos的3个头颅也代表认证过程中涉及的三方：Client、Server、KDC。

Kerberos是一种网络认证协议，它允许某实体在非安全网络环境下，向另一个实体以一种安全的方式证明自己的身份。

Windows的AD域环境使用Kerberos来进行验证。

<!-- more -->

### 0x01 Long-term Key与Short-term Key

在 Security 领域中，有的密钥可能长期保持不变，比如你的密码，可能几年都不曾改变，这样的 Key 被称为 `Long-term Key`。

使用`Long-term Key`应该有以下原则：

> 1.被`Long-term Key`加密的数据包不应在网络中传输，或者说不要用`Long-term Key`加密数据包。原因在于如果加密的数据包被攻击者抓包截取，那么只要时间充足，密钥被爆破出来的风险是很大的。
>
> 2.`Long-term Key`不应该使用明文方式存储，好的方法是用摘要算法计算其Hash值，从而保存Hash值。因为我们知道Hash算法是不可逆的，且不同输入计算出来的Hash是不同的，因此理论上拥有Hash值是不可能逆向破解获得明文Key，除非进行Hash碰撞，即对Hash算法尝试不同的输入，然后将算法输出Hash值与我们要破解的Hash值对比，本质是暴力破解。

我们一般会使用 `Short-term Key` 来加密需要进行网络传输的数据。顾名思义，这种 Key 只在短时间内有效，因此即使被加密的数据包被黑客截获，等他把 Key 计算出来的时候，这个 Key 也早就已经失效了。

由此我们也引出一个问题：

我们知道加密通信数据应该使用`Short-term Key`，但是`Short-term Key`作为一个短期有效的密钥，其管理分发是一个问题，即通信双方该如何即安全又便捷的协商出一个只有双方知道的`Short-term Key`呢？

Kerberos认证的第一阶段和第二阶段其实就是协商出通信所需的`Short-term Key`的过程。

在Kerberos认证中，`Short-term Ke`被称为`Session Key`。而Windows用户的密码就是我们前面说的`Long-term Key`，它以`NTLM Hash`方式存储在服务器中。

### 0x02 KDC(Key Distribution Center)

KDC(Key Distribution Center)，即密钥分发中心，作为第三方信任机构为C/S提供认证。

担任KDC的角色在物理层面上与DC(Domain Controller)，即域控所属同一主机。

KDC负责管理票据、认证票据、分发票据，但是KDC不是一个独立的服务，它由以下部分组成：

> - AS（Authentication Service）：主要用于生成Client与TGS通信的`Session Key`，记`K(c,tgs)`以及TGT。
> - TGS（Ticket Granting Service）：主要用于生成Client与Server通信的`Session Key`，记`K(c,s)`以及Tiket。

AD(Account Database)，它作为账户管理数据库，用与存储用户认证信息，即密码的`NTLM Hash`等。

### 0x03 Kerberos认证大致流程

![](02-Windows认证之Kerberos\1070321-20180417175541960-360210611.png)

第一阶段（AS Exchange）：生成一个用于加密Client与TGS的通信的`Session Key`。

> 生成`K(c,tgs)`的两份Copy，一份加密用于给Client，一份加密为TGT用于给TGS（但先发给Client，让其发给TGS）。

第二阶段（TGS Exchange）：生成一个用于加密Client与Server通信的`Session Key`。

> 生成`K(c,s)`的两份Copy，一份加密用于给Client，一份加密为Tiket用于给Server（但先发给Client，让其发给Server）。

第三阶段（CS EXchange）：Client与Server之间验证并使用`K(c,s)`加密通信。

### 0x04 Kerberos认证第一阶段：Authentication Service Exchange

![](02-Windows认证之Kerberos\QQ截图20190403201648.png)

**KRB_AS_REQ**

首先，客户端发送请求给KDC AS，请求消息包含以下三部分：

> - `Pre-authentication data`：用以证明Client身份的信息，它的内容是一般是被Client的`NTLM Hash`加密的 `Timestamp`。
> - `Client name & realm`：Client自身信息，简单地说就是 `DomainName\Username`，KDC AS用其查找AD数据库看用户是否存在。
> - `Server Name`：注意这里的ServerName并不是Client实际想要通信的Server，而是KDC TGS服务器的名称。

**KRB_AS_REP**

KDC收到请求消息后，根据提供的用户名在`AD(Account Database)`中寻找是否存在。

若存在，则将产生一个`Session Key`，记`K(c,tgs)`，并且将其生成两份Copy，分别用于给Client和KDC TGS。

对于给Client的那份`Session Key`，KDS AS会从`AD(Account Database)`中获取Client的`NTLM Hash`对其进行加密。

对于给KDC TGS的那份`Session Key`，KDS AS会从`AD(Account Database)`中获取`krbtgt`用户的`NTLM Hash`对其及其它信息进行加密，称为TGT（Ticket Granting Ticket）。

> - TGT中除了`Session Key`，还包括一些Client的用户名（DomainName\Username）、End time（TGT 到期的时间）等信息。
> - `krbtgt`用户是在新建一台域控制器时，由系统自动创建使得，用于Kerberos认证用的。因此TGT只有KDC能解密。

虽然产生的TGT是用于给KDC TGS的，但是KDC还是会把两份都发给Client，KDC TGS那份由Client发给KDC TGS。之所以这样做的目的是：

> - 首先Server不用维护一张庞大的会话密钥列表来应付不同的Client的访问，降低了Server的负荷；
> - 其次避免出现因为网络延时，Client的认证请求比Server的会话密钥早到达Server端，进而导致认证失败的情况。

### 0x05 Kerberos认证第二阶段：Ticket Granting Service Exchange

![](02-Windows认证之Kerberos\QQ截图20190403202909.png)

**KRB_TGS_REQ**

根据前面我们可以知道，此时Client拥有两份加密的`Session Key`，分别是

> - 用自己`NTLM Hash`加密的`Session Key`。
> - 用`krbtgt`用户的`NTLM Hash`加密的TGT。

首先，Client会使用自己的`NTLM Hash`解密属于自己的那份，解密后获得`Session Key`。

接着，Client会生成鉴别码（Authenticator），并使用`Session Key`对鉴别码进行加密。

> 鉴别码实际主要是Client的信息（DomainName\Username）、ServerName(DomainName\Server)以及当前时间的时间戳。
>
> 此处的ServerName是Client真正想访问的Server。
>
> 鉴别码的作用主要是为了证明该消息是Client自己发的。

最后，Client将加密的鉴别码与TGT发给KDC TGS。

**KRB_TGS_REP**

KDC TGS收到消息后，先使用自己（krbtgt）的`NTLM Hash`对TGT进行解密，获得`Session Key`、Client信息、TGT过期时间等信息。

然后使用`Session Key`解密被Client加密的鉴别码，获得Client信息、时间戳等信息。

比对两次解密得到的时间戳，确保在可允许的范围。

时间同步的重要性：

> 我们知道不管是Session Key还是票据都是有时效性的，TGT通常是8个小时，时效性的判断主要是用数据包传递的时间戳（Timestamp）与本地的时间做比较，因此Client、Server、KDC三者的时间同步是很重要，否则可能会造成验证失败，通常它们都需要配置从同一时间服务器同步时间。

验证通过后，KDC TGS会生成一个`Session Key`，记`K(c,s)`，该`Session Key`用于给Client和Server通信使用。

将`K(c,tgs)`加密`K(c,s)`，用于给Client。

同时也会生成一个Tiket用于给Server，并用Server的`NTLM Hash`加密该Tiket。Tiket包含：

> - 用于给Client和Server通信使用`Session key`，即`K(c,s)`。
> - Client的一些信息，如用户名。
> - Tiket过期时间。

可以发现这个阶段与上一个阶段是类似，KDC同样会把这两份加密了的`Session Key`都发给Client。

### 0x06 Kerberos认证第三阶段：Client/Server Exchange

**KRB_AP_REQ**

此时Client同样拥有两份加密的`Session Key`，`K(c,s)`。分别是

> - 用自己`NTLM Hash`加密的`Session Key`。
> - 用Server的`NTLM Hash`加密的Tiket。

接着Client会使用自己的`NTLM Hash`解密属于自己的那份，解密后获得`K(c,s)`。

使用`K(c,s)`加密鉴别码，将加密的鉴别码同Tiket发给服务器。

**KRB_AP_REP**

Server收到消息后，先使用自己的`NTLM Hash`对Tiket进行解密，获得`Session Key`、Client信息、TGT过期时间等信息。

然后使用`Session Key`解密被Client加密的鉴别码，获得Client信息、时间戳等信息。

过程与`KRB_TGS_REP`阶段差不多，不再赘述。

但有一点是，

如果Client需要进行双向验证，Server从鉴别码中提取时间戳，使用`K(c,s)`进行加密，并将其发送给Client用于Client验证Server的身份。

### 白银票据（Silver Tiket）

通过前面我们已经知道Kerberos的认证大致流程，在第三阶段认证的`KRB_AP_REQ`时，Client拥有两份加密的`Session Key`，`K（c,s）`分别是：

> - 用自己`NTLM Hash`加密的`Session Key`。
> - 用Server的`NTLM Hash`加密的Tiket。

Tiket只有Server可以解密，这是因为Tiket是使用Server的`NTLM Hash`进行加密的。但是这也意味着如果我们拥有Server的Hash，那么意味着我们可以解密以及伪造Tiket，从而绕过KDC直接进行验证。

### 黄金票据（Golden Tiket）

通过前面我们已经知道Kerberos的认证大致流程，在第二阶段认证的`KRB_AS_REQ`时，Client拥有两份加密的`Session Key`，`K（c,tgs）`分别是：

> - 用自己`NTLM Hash`加密的`Session Key`。
> - 用`krbtgt`用户的`NTLM Hash`加密的TGT。

前面我们说过，TGT只有KDC可以解密，这是因为TGT是使用`krbtgt`用户的`NTLM Hash`进行加密的，而该Hash只有KDC知道。但是这也意味着如果我们拥有`krbtgt`用户的Hash，那么意味着我们可以解密以及伪造TGT，

**参考资料：**

https://blog.csdn.net/lengxiao1993/article/details/20458809

https://payloads.online/archivers/2018-11-30/1?tdsourcetag=s_pcqq_aiomsg#%E5%9F%9F%E8%AE%A4%E8%AF%81%E4%BD%93%E7%B3%BB---kerbroes

https://klionsec.github.io/2016/08/10/ntlm-kerberos/?tdsourcetag=s_pcqq_aiomsg

https://blog.csdn.net/wulantian/article/details/42418231