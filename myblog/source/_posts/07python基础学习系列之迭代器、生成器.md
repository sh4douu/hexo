---
title: Python基础学习系列Chapter 8：迭代器与生成器
date: 2019-03-29 15:19:48
tags:
	- Python基础
	- Language
categories: Python
---

#### 0x00 概述

- 可迭代对象：可用迭代工具（如for循环）迭代一次产生一个结果的对象。
- 迭代器：可以记住取值的状态，一次返回一个值。
- 可迭代对象执行`__iter__`方法，或者对应的iter()内置方法后，会返回一个迭代器。
- 迭代工具：for、列表解析、in、map()等。

<!-- more -->

#### 0x01 可迭代对象

- 支持`__iter__`或`__getitem__`方法的对象就是可迭代对象。
- 可迭代对象不等于迭代器，但是可迭代对象可以产生迭代器。
- 可迭代对象可以多次打开（生成）迭代器。
- 可迭代对象需要迭代器才能一次取一个值。
- for循环的实质也是先将可迭代对象先转换为迭代器。

##### 0x02 可迭代对象产生迭代器

- 调用内置方法iter()或重载方法`__iter__`后会返回一个迭代器。

```python
# 列表是可迭代对象，但不是迭代器
list=[1,2,3,4,5,6]

# 两种方法
iter(list) 
list.__iter__()
```

- 字典、元组、字符串、集合都是可迭代对象，但不是直接的迭代器。

##### 0x03 判断是否为可迭代对象

```python
from collections import Iterable

isinstance([], Iterable) #True
isinstance({}, Iterable) #True
isinstance('abc', Iterable) #True
```

#### 0x04 迭代器

- 支持`__next__`方法的对象就是迭代器。
- 迭代器可以记住状态并每次返回一个值。
- 取到最后一个值时会抛出一个StopIteration异常,依靠该异常可以判断到了边界。

```python
list=[1,2,3]
iterator = iter(list)

iterator.__next__() #返回1
iterator.__next__() #返回2
iterator.__next__() #返回3
iterator.__next__() #抛出异常StopIteration

# iterator.__next__()等同于 next(iterator)
```

##### 0x05 判断是否为迭代器

```python
from collections import Iterator

isinstance([], Iterator) #False
isinstance((x for x in range(10)), Iterator) #True
```

#### 0x06 迭代工具 for

- for循环的实质是：先将可迭代对象用iter()方法将其变为迭代器，再用迭代器的`__next__`方法逐一取值，最后捕获StopIteration异常判断结束。

```python
# for i in [1, 2, 3, 4, 5]
it = iter([1, 2, 3, 4, 5])
while True:
    try:
        # 获得下一个值:
        i = it.__next__()
    except StopIteration:
        # 遇到StopIteration就退出循环
        break
```

#### 0x07 生成器 generator

- 生成器函数编写使用常规def语句，与普通函数区别在于：生成器函数yield一个值，而不是return一个值。
- 生成器函数：使用了yield语句的函数。可以每次返回一个值并可以从上次退出的地方继续的函数。
- yield将函数状态挂起，并向调用者返回一个值，没指定则返回none。挂起的状态包含整个本地作用域。
- 调用生成器函数会返回一个生成器，生成器可以用迭代工具迭代。
- 生成器本身也是一个迭代器（有`__next__`），而且是单迭代器对象，虽然有`__iter__`方法，但是没用，不能产生多个生成器。
- 第一次执行`__next__（）`会从函数开始执行完全的一遍生成器函数，之后的next会从yield处开始。

```python
def square(num): #定义生成器函数
    for i in range(num):
        yield i**2
res = square(5) #生成生成器

for i in res: #迭代生成器
    print(i)

```

##### 0x08 表达解析式生成生成器

```python
# 用圆括号
(i*2 for i in range(11))
```

##### 0x09 send()：发送消息给生成器并返回发送的内容。

- 会等同于`__next__`方法一样工作。
- 还可以提供调用者与生成器之间的通信，即调用者可以用该方法发送信息给生成器。
- 发送的消息由yield接收并返回，因此yield表达式可以写为：X=yield Y。yield每次会返回Y，同时也会接收send()发来的消息并赋值给X。
- 遇到send()和yield都会进行中断。

```python
# 实现协程并行
# 生产者与消费者模型
import time
def consumer(name):
    print("%s 准备吃包子啦!" %name)
    while True:
       baozi = yield
       print("包子[%s]来了,被[%s]吃了!" %(baozi,name))

def producer(name):
    c1 = consumer('A')
    c2 = consumer('B')
    c1.__next__()
    c2.__next__()
    print("老子开始准备做包子啦!")
    for i in range(10):
        time.sleep(1)
        print("做了2个包子!")
        c1.send(i)
        c2.send(i)
producer("老子")
```

