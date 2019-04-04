---
title: Windows认证之NTLM
date: 2019-04-03 11:58:18
tags:
	- NTLM
	
categories: 内网渗透
---

### 0x00 Windows密码存储

We all konw，明文存储密码是很愚蠢的行为，因为如果系统被攻破，明文密码被拿到，那么意味就全完了，但是如果是先加密再进行存储的话，即使黑客拿到加密的密码，破解也是需要费很大功夫和时间的。

而现在采用的加密算法基本都是Hash摘要算法，Hash算法可以把任意长度的输入通过散列算法输出一串固定长度的值。

常见的Hash算法诸如md5、SHA等，Hash算法有如下特点：

> - 单向不可逆性：通过Hash值无法反向推导出输入的值。
> - 如果两个哈希值相同，两个输入值很可能(极大概率)是相同的，但也可能不同，这种情况称为“哈希碰撞”

说了那么多就是为了表明一点，Windows在存储用户密码时也不直接存储明文密码，存储的是明文密码经过Hash计算得到的Hash值，Windows存储的Hash称为`NT Hash`或`LM Hash`（现在基本都为`NT Hash`）。

`NT Hash`与`LM Hash`是如何计算得到的，我们放在下节讨论，首先用户的Hash肯定需要存储在系统的某个文件或数据库中，Windows将用户密码计算得到的Hash存储在：

> - 工作组或本地环境中，存储在一个名为`SAM`的文件内，文件路径为：`%SystemRoot%\system32\config\sam`
> - 域环境中，存储在NTDS数据库中。

<!-- more -->

### 0x01 NT Hash与LM Hash

`LM Hash`是早期Windows存储的Hash格式，从Windows Vista/Server 2008开始默认是关闭的。

`NT Hash`，又叫`NTLM Hash`，或者直接叫`NTLM`(后面带v1/v2才是协议),是现在Windows主要的存储格式。

**NT Hash算法：**

1.先将用户密码转换为十六进制格式。

2.将十六进制格式的密码进行Unicode编码。

3.使用MD4摘要算法对Unicode编码数据进行Hash计算，得到结果就是`NT Hash`。

```
admin -> hex(16进制编码) = 61646d696e
61646d696e -> Unicode = 610064006d0069006e00
610064006d0069006e00 -> MD4 = 209c6174da490caeb422f3fa5a7ae634
```

破解方式：

```
john --format=nt hash.txt
hashcat -m 1000 -a 3 hash.txt
```

**LM Hash算法：**

```
将所有小写字母转换为大写字母
>123ABC    // 未达到7个字符

将密码转化为16进制，分两组，填充为14个字符,空余位使用0x00字符填补
>31323341424300000000000000

将密码分割为两组7个字符
>31323341424300 00000000000000   //16进制

将每组转化为比特流，不足56Bit则在左边加0
>31323341424300 ->(转换为二进制) 110001001100100011001101000001010000100100001100000000-> (补足56Bit) 00110001001100100011001101000001010000100100001100000000

将比特流按照7比特一组，分出8组，末尾加0
由于后者都为0，结果可想而知，那就都是0;
将每组比特流转换为16进制作为被加密的值，使用DES加密，字符串 “KGS!@#$%”为Key(0x4B47532140232425)，得到8个结果 ，每个结果转换为16进制。
-> 00110000100110001000110001101000000101000001001000001100 00000000
->30988C6814120C00 -> DES(30988C6814120C00) -> 48-D7-EB-91- 2F-5E-69-7C

由于我们的密码不超过7字节，所以后面的一半是固定的:
AA-D3-B4-35-B5-14-04-EE

连接两个DES加密字符串。得到LM哈希。
48-D7-EB-91-2F-5E-69-7C-AA-D3-B4-35-B5-14-04-EE
```

通过算法我们可以看到`LM Hash`脆弱点就在于DES的Key（`KGS!@#$%`）是固定的，也就是说，有了Key就能够解出原文。

并且根据`LM Hash`特征，也能够判断用户的密码是否是大于等于7位。

破解方式：

```
john --format=lm hash.txt
hashcat -m 3000 -a 3 hash.txt
```

### 0x02 NTLM v1/v2（AKA Net-NTLM v1/v2）

> `NTLM（NT LAN Manager）`，又称为`Net-NTLM`，是一个网络认证协议，他是一种用于在Client和Server之间进行Challenge/Response验证的协议。

**NTLM v1/v2与LM、NT Hash的关系：**

NTLM是一个认证协议，使用LM Hash或NT Hash来进行验证。

> - 在NTLM v1中，使用LM Hash和NT Hash来进行验证，如今v1已被废弃。
> - 自从Windows 2000之后，NTLM v2是默认的认证协议。

NTLM v1与NTLM v2之间的区别我们放在网络认证的章节来讨论。

### 0x03 本地认证

> 所谓的本地认证即在物理主机上直接进行登录的认证过程。

**本地认证的大致流程：**

> winlogon.exe -> 接收用户输入 -> lsass.exe -> (认证)

- Windows Logon Process(即 winlogon.exe)，是Windows NT 用户登 陆程序，用于管理用户登录和退出。
- LSASS用于微软Windows系统的安全机制。它用于本地安全和登陆策略。

**具体流程：**

1.当我们刚开机、注销等操作后，`winlogon.exe`进程会显示一个登录界面要求我们输入用户名和密码。

2.我们输入用户名和密码后，会被`winlogon.exe`获取，然后将其发送给`lsass.exe`进程。

3.`lsass.exe`将明文密码计算得到`NT Hash`（不考虑LM）。

4.之后会将用户名和计算得到的`NT Hash`拿到`SAM`数据库去查找比对。

### 0x04 网络认证

> 网络认证，顾名思义即通过网络进行认证，常见于工作组环境中，例如访问共享时进行的认证就是网络认证。网络认证使用的协议为NTLM协议。

**NTLM v2认证流程：**

认证流程大致可以分为三步：

> - 协商：主要用于确认双方协议版本。
> - 质询：进行挑战（Chalenge）/响应（Response）认证机制的阶段，是最重要的阶段，也是我们主要讨论的环节。
> - 验证：质询完成后，验证结果。

![](01-Windows认证之NTLM\QQ截图20190404133112.png)

质询的完整过程：

客户端向服务器端发送包含用户信息(用户名)的请求。

服务器接受到请求后，先使用接收到用户名到`SAM`数据库中查找看是否存在，若存在则提取对应的`NT Hash`。

然后服务器会生成一个16位的随机数，称之为“Challenge”，并使用提取的`NT Hash`加密`Challenge`(16位随机字符)，得到加密结果记作`Challenge2`。同时，将Challenge(16位随机字符)发送给客户端。

客户端接受到`Challenge`后，使用将要登录的账户对应的`NT Hash`加密`Challenge`生成`Response`(将明文密码计算为`NT Hash`是客户端计算机接收到用户输入的密码后自动做的事情)，然后将`Response`发送至服务器端。

验证： 服务器端收到客户端的`Response`后，比对`Chanllenge2`与`Response`是否相等，若相等，则认证通过。

**NTLM v1/v2区别：**

认证的流程是一样的，不同的地方在于：

> - Challage长度：NTLM v1的Challenge有8位，NTLM v2的Challenge为16位。
> - 加密Challenge的算法：NTLM v1的主要加密算法是DES，NTLM v2的主要加密算法是HMAC-MD5。

### 0x05 Pass The Hash

观察NTLM认证的过程可以发现，整个过程中实际上并没有进行明文密码的传递，也没有使用明文密码来加密Challenge，而是用明文密码对应`NT Hash`来加密`Challenge`，那么就意味着：

> 只要我们知道相应用户密码的`NT Hash`，我们就可以自己构造出`Response`发送给服务器，从而通过验证。我们并不需要知道用户的明文密码。这种攻击方式就称之为哈希传递（Pass The Hash）。

可以实现哈希传递攻击的攻击的工具：

- Smbmap
- Smbexec
- Metasploit
- CrackMapExec

> cme smb 192.168.3.5 -u administrator -H dab7de8feeb5ecac65faf9fdc6cac3a9 -x whoami

**参考资料**

https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4

https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

http://www.adshotgyan.com/2012/02/lm-hash-and-nt-hash.html

https://payloads.online/archivers/2018-11-30/1?tdsourcetag=s_pcqq_aiomsg#0x00-%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81

https://github.com/crazywa1ker/DarthSidious-Chinese/blob/master/getting-started/intro-to-windows-hashes.md