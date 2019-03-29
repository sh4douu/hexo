---
title: Python基础学习系列Chapter 10：装饰器
date: 2019-03-29 16:11:48
tags:
	- Python基础
	- Language
categories: Python
---

### 装饰器

##### 0x00 概述

> 装饰器本质上是一个 Python 函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。 
>
> 它经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景 。
>
> 概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。 

<!-- more -->

##### 0x01 简单装饰器

- 执行装饰函数(log())，并将被装饰函数(now())作为一个参数传给装饰函数。然后将装饰函数执行的返回值赋值给被装饰函数名。

```python
def log(func):  #装饰函数
    def wrapper():
        print('Execute Function: %s' % func.__name__)
        return func()
    return wrapper

@log  #等同于now = log(now)

def now():  #被装饰的函数
    print('Execute Result : 2018-8-1')

now()  #相当于执行了wapper()
print(now.__name__) #返回wrapper ，而不是now。因为执行了now = wrapper。
```

##### 0x02 带参数的业务函数（被装饰函数）

- 使用*args 和 **kwargs

```python
def log(func):
    def wrapper(*args, **kw):
        print('Execute Function: %s' % func.__name__)
        return func(*args, **kw)
    return wrapper

@log  #等同于now = log(now)

def now(tag):
    print('Execute Result : 2018-8-1  %s'%tag)

now('tag1')  #相当于执行了wapper('tag1')
print(now.__name__) #返回wrapper ，而不是now。因为执行了now = wrapper。
```

##### 0x03 装饰器进阶：带参数的装饰器

```python
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

@log('execute')  #等同于now = log('execute')(now),先执行后传函数
def now():
    print('2015-3-25')

now()
print(now.__name__)  #返回wrapper ，而不是now。因为执行了now = wrapper。
```

> 先执行 log('execute') ，返回decorator函数；之后再像普通装饰器一样，执行decorator(now)，返回wrapper函数，并将wrapper函数赋值给now变量。

- 总结：
  - 假设：log函数为装饰器函数，now函数为普通函数。
    - @log相当于 now=log(now)
    - @log()相当于 now=log()(now)
    - @log(“test”)相当于now=log(“test”)(now)

#### 0x04 装饰器标准写法

- 可以看到，前面的例子中，最后now函数的函数名（now.\__name__）都变成了wrapper，这样可能会导致某些依靠判断函数名的语句出错。
- 因此，可以通过导入functools模块，并使用@functools.wraps(func)解决，而不用使用语句：`now.__name__=func.__name__ `
- 普通装饰器:

```python
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

@log
def now(tag):
    print('Execute Result : 2018-8-1  %s'%tag)

now('tag1')  #相当于执行了wapper('tag1')
print(now.__name__) #返回now,而不是wrapper。
```

- 带参数的装饰器：

```python
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

@log("execute")

def now(tag):
    print('Execute Result : 2018-8-1  %s'%tag)

now('tag1')  #相当于执行了wapper('tag1')
print(now.__name__) #返回now,而不是wrapper。
```



