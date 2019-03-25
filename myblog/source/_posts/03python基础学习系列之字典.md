---
title: Python基础学习系列Chapter 4：字典
date: 2019-03-19 16:12:43
tags:
	- Python基础
	- Language
categories: 语言
---

### 0x00 字典概述

- 任意对象的无序集合
- 可任意嵌套
- 属于可变类型对象 
- 字典存储0个或多个对象的引用
- 通过key而不是下标取值，且key具有唯一性

<!-- more -->

### 0x01 字典创建

```python
dic = {}
dic = {'name':'opaque','job':'hacker'}
dic = dict(name='opaque',job="hacker")
```

### 0x02 字典操作

#### **查相关**

##### 索引键

```python
dic = {'name':'opaque','job':{'IT':hacker'}}

dic['name']        #索引key为name对应的值，返回'opaque'
dic['job']['IT']   #索引嵌套的内容，返回'hacker'
```

##### get() #根据键返回值

```python
dic.get('name')
dic.get('job').get('IT')
dic.get('other','abc')     #不存在‘other’键时，返回‘abc’
```

- get()与索引的区别在于，索引不存在的键时会引发异常错误，而get()不存在的键时，默认会返回NONE，也可自己设置（如上面第三个栗子）。

##### in #成员关系测试

```python
dic = {'name':'opaque','job':{'IT':hacker'}}

'name' in dic     #测试字典是否存在key'name'
```

##### items() #列出所有键值对

```python
dic.items()
```

- 返回的结果是形似以元组为元素的列表的可迭代对象

##### keys() #列出所有键名列表

```python
dic.keys()
```

- 返回形似列表的可迭代对象

##### value() #列出所有值列表

```python
dic.values()
```

- 返回形似列表的可迭代对象

#### **增相关**

##### 赋值

```python
dic={}

dic['age']=18                 #新增一个key为'age',value为18
dic['other']={'sex':'man'}    #嵌套赋值
```

- 已有则修改，无则新增

##### update() #合并字典

```python
dic1 = {'name':'opaque'}
dic2 = {'job':'hacker'}

dic1.update(dic2)      #将dic2合并至dic1
```

##### setdefault() #增或查

```python
dic = {'name':'opaque'}
dic.setdefault('name',18)               #已存在key'name',直接返回value'opaque'
dic.setdefault('age',18)                #新增
dic.setdefault('job',{'IT':'hacker'})   #新增
```

- 字典中已经存该key名，则返回对应的value，若key不存在，则插入key以及value。

#### **改相关**

##### 赋值修改

```python
dic={'job':'hacker'}

dic['job']='HACKER'
```

#### **删相关**

##### pop() #删除指定key的项

```python
dic = {'name':'opaque','job':'hacker'}

dic.pop('name')
```

- 与列表的pop()差不多，会返回删除的key对应的value

##### popitem() #随意删除一个项

```python
dic.popitem()
```

- 返回以被删除的键值作为元素的元组

##### del

```python
dic = {'name':'opaque','job':'hacker'}

del dic['name']
```

#### **其他**

##### fromkeys() #初始化一个字典

```python
dic = fromekeys(['name','age'])             #创建一个字典，key分别为'name'、'age',value都为NONE
dic = fromekeys(('name','age'),'opaque')    #value都为'opaque'
```

- 第一个参数为序列对象，序列中的元素作为key
- 第二个参数为value，序列中的全部key共享引用该value。
- 注意该value不要在嵌套，否则引用会有问题：

```python
dic = fromekeys(('name','age'),{'a':'opaque'})

dic[name]['a']=2     #会导致age的a的值也会变，因为共享引用
```

#### **字典遍历**

##### 三种方法

```python
dic = {'name':'opaque','job':'hacker'}

for k in dic:
    print(k,dic[k])
    
for k,v in dic.items():
    print(k,v)
    
for k in dic.keys():
    print(k,dic[k])

```

- 优先第一种，第一种比第二种效率高，第二种有类型转换的过程。

#### 二次总结

```python
1.字典是任意对象的无序集合。
2.字典以键值对的方式存储。
3.字典是可以原处修改的对象，存储的是对象的引用。
4.强制类型转换：dict()
5.常量：
	dict={'k1':'v1'}
6.操作：
	查：
		索引通过键索引，而不是下标：dict['key']
		get() #根据键返回值,对比索引差别在于取不存在的键不会报错。
		items()  #列出所有键值对
		keys()   #列出所有键名列表
		values() #列出所有值列表
		
	增：
		索引不存在的键并赋值是新增一个元素。
		update() #用一个字典新增到另一个字典中。（原处修改，同列表的extend()）
		
	改：
		索引已存在的键并赋新值是修改。
	
	删：
		pop()  #删除指定key的项并返回值。（原处修改）
		popitem() #随意删除一个项并返回键值。（原处修改）
		clear()	 #清空字典
		del dict['key'] #删除索引。（原处修改）
		
6.遍历字典：
		for k in dict:
			 print(k,dict['k'])
		
		for k,v in dict.items():
			print(k,v)
			
		for k in dict.keys():
			print(k,dict['k'])

```

