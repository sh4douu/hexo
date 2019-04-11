---
title: Python模块 Chapter 3：subprocess
date: 2019-04-10 10:49:30
tags:
	- Python模块
	- Language
categories: Python
---

### 概述

`subprocess`模块允许你创建一个新的进程，新建进程即当前程序的子进程。可以连接到新建进程的input/output/error pipes, 并且可以获取到它们的返回码。

这个模块是为了取代下面这些比较老的模块功能：

```python
os.system
os.spawn*
```

subprocess模块提供两种方法来创建新的进程：

- subprocess.run()
- subprocess.Popen()

> 使用`run()`方法创建新进程可以满足绝大多数需求，但`Popen（）`类提供比`run()`更高级更灵活的用法。

