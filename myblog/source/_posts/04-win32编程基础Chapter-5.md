---
title: Win32编程基础Chapter 5：分支和循环
date: 2019-04-02 09:01:49
tags:
	- Win32
	- 汇编语言
categories: Win32
---

#### 0x00 分支语句

语法格式：

```assembly
.if 条件表达式
	code

[.elseif 条件表达式]
	code

[.else]
	code

.endif
```

<!-- more -->

源程序：

![](04-win32编程基础Chapter-5\QQ截图20190402090659.png)

编译后反编译：

![](04-win32编程基础Chapter-5\QQ截图20190402091215.png)

![](04-win32编程基础Chapter-5\QQ截图20190402091227.png)

#### 0x01 循环语句

语法格式：

```assembly
.while 条件表达式
	code
	[.break [.if 条件表达式]]
	[.continue]
.endw
```

源程序：

![](04-win32编程基础Chapter-5\QQ截图20190402091959.png)

编译后反汇编：

![](04-win32编程基础Chapter-5\QQ截图20190402092028.png)