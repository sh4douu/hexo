---
title: Python基础学习系列Chapter 9：内置方法
date: 2019-03-29 15:56:48
tags:
	- Python基础
	- Language
categories: Python
---

### A

#### abs() #绝对值

##### all() #序列的元素全真为真(负数也算真)

##### any() #序列的元素任一为真则真

##### max() #方法返回给定参数的最大值，参数可以为序列。

##### min() #方法返回给定参数的最小值，参数可以为序列。

##### pow() #方法返回 xy（x的y次方） 的值。

##### sum() #对可迭代对象求和。

##### divmod(a, b) #给定被除数和除数，返回商和余数在元组中。

<!-- more -->

------

##### ascii() #类似 repr() 函数,返回一个表示对象的字符串

------

##### bin() #返回十进制数的二进制

##### oct() #函数将一个整数转换成8进制字符串。

##### hex() #函数用于将10进制整数转换成16进制整数。

------

### B

##### bool() #返回给定对象的布尔值

##### bytearray() #返回一个新字节数组。数组里的元素是可变的

##### callable() #检查一个对象是否是可调用的。

> 对于函数, 方法, lambda 函式, 类, 以及实现了` __call__` 方法的类实例, 它都返回 True。

------

##### chr() # 返回值是当前整数对应的ascii字符。

##### ord() # 返回字符对应的十进制整数。

------

##### compile() #将一个字符串编译为字节代码。

------

##### eval() #执行一个字符串表达式，并返回表达式的值。

##### exec() #执行储存在字符串或文件中的 Python 语句。

- 相比于 eval，exec可以执行更复杂的 Python 代码。
- exec 返回值永远为 None。

------

##### frozenset() #返回一个冻结的集合，冻结后集合不能再添加或删除任何元素。

------

##### enumerate()

- 给定一个可迭代对象返回一个迭代器。迭代器每次可返回可迭代对象的元素及其元素对应的下标在一个元组中。

##### filter() #函数用于过滤序列，过滤掉不符合条件的元素，返回由符合条件元素组成的迭代器。

- 迭代序列，将每次迭代结果作为参数传入函数中。
- 返回一个迭代器。

```python
for i in filter(lambda x:x>5,range(10))
# 返回 6,7,8,9
```

##### map() #迭代序列的每个结果作为参数传入函数中，执行函数的返回值放入一个迭代器。

- 返回一个迭代器。

```python
a = map(lambda x:8 if x>2 else x,range(4))
for i in a:
    print(i) #返回0，1，2，8
```

##### zip() #分别迭代序列，并将每次结果先放入一个元组，最终放入迭代器。

- 返回一个迭代器。

```python
a = [1,2,3]
b = [4,5,6]
for i in zip(a,b) #返回(1, 4), (2, 5), (3, 6)
```

------

##### globals() #以字典类型返回当前位置的全部全局变量。

##### locals() #函数会以字典类型返回当前位置的全部局部变量。

##### hash() #获取取一个对象（字符串或者数值等）的哈希值。

------

##### dir()

- 函数不带参数时，返回当前范围内的变量、方法和定义的类型列表；
- 带参数时，返回参数的属性、方法列表。

##### id() #函数用于获取对象的内存地址。

- == 判断字符是否对应相同，相同返回True
- is 判断内存地址是否相同，相同返回True

##### type() #返回对象的类型。

- isinstance() 与 type() 区别：type() 不会认为子类是一种父类类型，不考虑继承关系。

##### isinstance() #判断对象是否为另一对象的实例。

- isinstance() 与 type() 区别：isinstance() 会认为子类是一种父类类型，考虑继承关系。

##### issubclass() #判断对象是否为另一对象的子类。

##### len() #返回对象（字符、列表、元组等）长度或项目个数。

------

##### memoryview() # 返回给定参数的内存查看对象(Momory view)。

##### range() 函数可创建一个整数列表，一般用在 for 循环中。

```python
range(start, stop[, step])
```

##### reversed() #返回一个反转的迭代器。

##### round() #方法返回浮点数x的四舍五入值。

##### sorted() 函数对所有可迭代的对象进行排序操作。

- sort 与 sorted 区别：

> sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
>
> sort是原处修改，sorted是产生新对象。

```python
# 按value进行排序
sorted（dict.items(),key=lambda x:x[1]）
```

##### vars() #函数返回对象object的属性和属性值的字典对象。

##### \_\_import__()  #用于动态导入模块 。 

- 即可以以字符串类型的模块名导入模块。

```
import os <==> __import__('os')
```

###### 反射

```python
1.反射：利用字符串的形式去对象中操作（获取，查询，设置，删除）属性.
	getattr(module,attr)：到module对象（可以是模块等）查询attr属性并返回（可以是函数等）。
	hasattr(module,attr)：判断module对象是否有attr属性，有返回True,没有false。
	setattr(module,attr)：到module对象设置属性（导入模块会存在内存中，只会影响内存中的模块。）
	delattr(module,attr)：删除module对象的attr属性（在内存中而不会影响原模块 ）。

2.例子：例如通过反射实现用户输入URL自动跳转到相应页面。
	import modul
	def f1():
    	user_input = input('please input url:')
    	if hasattr(modul,user_input):
        	fun=getattr(modul,user_input)#返回函数体（函数名），而不是执行结果
        	fun()
    	else:
        	print('not found')
	f1()


#当需要的属性处于不同对象（模块）时，需要导入很多模块，不现实
#可以通过根据用户输入的判断导入哪个模块及其属性
#用户输入类型为str,但import module  ！=  import ’module‘
#但mod=__import__('mudule')等价于import module as mod
#提供了通过字符串导入模块的功能
	user_input=input('please input url(xx/xx):')#输入格式为【模块名/属性】
	def f2():
    	mod_name,attr_name=user_input.split('/')
    	modu=__import__(mod_name) #引入模块命名为modu
    	if hasattr(modu,attr_name):
        	fun=getattr(modu,attr_name)
        	fun()
    	else:
        	print(404)
	f2()

#若要导入模块不在当前目录下，而是某目录下（如abc目录下）
#__import__（'abc.module'）导入只会到abc，而不会到module
#__import__（'abc.module'，fromlist=True）加了该参数便可以
```

