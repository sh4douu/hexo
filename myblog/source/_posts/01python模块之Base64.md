---
title: Python模块 Chapter 1：Base64
date: 2019-03-29 16:22:48
tags:
	- Python模块
	- Language
categories: Python
---

### Base64

#### 0x00 Base64编码概述

- Base64字符集共64个ASCII字符，Base64编码即用这64个ASCII字符来表示任意二进制数据的方法。
- Base64主要是为了解决有些非打印字符二进制无法正常显示。比如我们使用notepad打开图片，会有乱码。通过对任何二进制数据进行Base64编码后，该数据都可以以字符的形式表现。

<!-- more -->

#### 0x01 编码原理

1. 将内存中每3个字节的二进制数据分为一组，一组3*8=24个字节。
2. 再将每组分为4个小组，每个小组24/4=6bit。
3. 计算每个小组的数字索引（0——63）。
4. 将计算得到的数字索引查找Base字符集表得到对应ASCII字符，该字符就是该小组编码后得到的结果。
5. 以此类推，依次进行分组编码。

![](01python模块之Base64\QQ截图20190329170758.png)

**Misc**

- 所以，Base64编码会把3字节的二进制数据编码为4字节的文本数据。
- 如果要编码的二进制数据不是3的倍数，Base64用`\x00`字节在末尾补足后，再在编码的得到的字符串末尾加上1个或2个=号，表示补了多少字节，解码的时候，会自动去掉。
- 可以自己定义64个字符的排列顺序，这样就可以自定义Base64编码，不过，通常情况下完全没有必要。
- Base64是一种编码方法，不能用于加密，不存在安全性，即使使用自定义的编码表也不行。
- Base64适用于小段内容的编码，比如数字证书签名、Cookie的内容等。
- 由于‘=’字符也可能出现在Base64编码中，但‘=’用在URL、Cookie里面会造成歧义，所以，很多Base64编码后会把=去掉。去掉=后怎么解码呢？

> 因为Base64是把3个字节变为4个字节，所以，Base64编码的长度永远是4的倍数，因此，需要加上=把Base64字符串的长度变为4的倍数，就可以正常解码了。

#### 0x02 Python Base64

```python
1.标准Base64:
	import base64

	base64.b64encode(b'superwu') #base64编码
	base64.b64decode(b'c3VwZXJ3dQ==') #base64解码

2.由于标准的Base64编码后可能出现字符+和/，在URL中就不能直接作为参数,会产生歧义，所以又有一种"url safe"的base64编码，其实就是把字符+和/分别变成-和_：
	>>> base64.b64encode(b'i\xb7\x1d\xfb\xef\xff') #标准base64编码
		b'abcd++//'
	>>> base64.urlsafe_b64encode(b'i\xb7\x1d\xfb\xef\xff') #
		b'abcd--__'
	>>> base64.urlsafe_b64decode('abcd--__')
		b'i\xb7\x1d\xfb\xef\xff'
```



