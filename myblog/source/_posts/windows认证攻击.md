---
title: Windows认证攻击
date: 2019-04-06 10:25:26
tags:
	- Pass The Hash
categories: 内网渗透
---

### 0x00 窃取Windows凭证

**Mimikatz**

我们在前面的文章已经提到过，当用户输入用户名密码登录时，用户名密码会被发送到`lsass.exe`进程中，然后`lsass.exe`会将明文密码计算为`NTLM Hash`，在用户登录期间一直保存在内存中。

Mimikatz是法国人Gentil Kiwi编写的一款Windows平台下的神器，它具备很多功能，其中最亮的功能是直接从`lsass.exe`进程里获取`Windows`处于active状态账号的明文密码或`NT Hash`。在Windows >= 8.1情况下，无法提取明文密码，但是可以提取到`NT Hash`。

[Mimikatz下载链接。](https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-delegation)

<!-- more -->

工具演示：

![](windows认证攻击\QQ截图20190407101243.png)

**LaZagne**

既然说到提取密码，那么这里不得不说一款神器，它可以用于用户提取用户计算机上的明文密码。例如浏览器记住的密码、WIFI密码等。具体支持提取的密码有：

![](windows认证攻击\softwares.png)

[LaZagne下载地址。](https://github.com/AlessandroZ/LaZagne)

工具演示：

![](windows认证攻击\QQ截图20190407105139.png)

### 0x01 Pass The Hash

在之前的文章中，我们已经知道了Windows在进行Challenge/Response网络验证时，使用的用户密码对应的`NT Hash`加密`Server Challenge`得到Response（验证过程中第三个数据包），因此，我们只要知道用户的`NT Hash`，就可以直接使用Hash向服务器发起认证，并不需要知道明文密码。这种攻击就称之为哈希传递（Pass The Hash）。如果对此还有疑问，请跳转[这里](https://sakuxa.com/2019/04/03/01-Windows%E8%AE%A4%E8%AF%81%E4%B9%8BNTLM/)学习NTLM网络认证。

常用实现Pass The Hash攻击的工具：

- Smbmap
- Smbexec
- Metasploit
- CrackMapExec

接下来我们使用最后两个工具来模拟进行哈希传递攻击。

**NTLM Hash**

在使用工具之前，有一点需要注意的是，我们知道早期Windows会存储用户密码对应的两种Hash值：`LM Hash:NT Hash`，直到Windows Vista和Window Server 2008之后，就只存储`NT Hash`了。为此，有些工具在给其提供Hash值参数时，需要使用`LM Hash:NT Hash`这样的格式，即使没有`LM Hash`，也要使用任意值进行填充以满足格式，通常的做法是填充32个0。但是也有的工具只需要给出`NT Hash`即可。Metasploit中的psexec和CrackMapExec就分别对应这两种情况。

**Meatasploit Psexec**

> Meatasploit中的`psexec`模块可以实现Hash传递攻击，攻击成功后会返回一个Meterpreter Session。

攻击演示：

- `use exploit/windows/smb/psexec`
- 设置smbpass的格式是`LM:NTLM`，但在w2k8中，LM已禁用，所以`LM`处可以随意用32个字符填充满足模块所需的格式即可，比如使用32个0。

![](windows认证攻击\QQ截图20190406101732.png)

**CrackMapExec**

> 使用Hash值进行哈希传递攻击，让目标计算机执行指定的命令（-x "cmd"）。

Kali安装CrackMapExec：

- `apt-get install crackmapexec`

使用格式：

```
crackmapexec  smb  <IP>  -u <Username>  -H <NT hash>  -x "cmd"
```

攻击演示：

使用Hash值进行攻击，让目标计算机执行`msg adminitrator hacked`命令，执行成功后，目标计算机administrator用户的桌面会弹出一个消息框。

![](windows认证攻击\QQ截图20190405101817.png)