---
title: Python基础学习系列Chapter 3：列表
date: 2019-03-19 15:37:28
tags:
	- Python基础
	- Language
categories: Python
---

### 0x00 列表概述

- 任意对象的有序集合。
- 可任意嵌套。
- 属于可变对象类型。
- 列表存储了0个或多个对象的引用。

<!-- more -->

### 0x01 列表创建

```python
list=[]
list=['opaque','super',1,3]
list=list('opaque')
```

### 0x02 列表操作

#### **查相关**

##### 索引：按下标索引元素

```python
list=['opaque','super',1,3]

list[0]    #返回'opaque'
list[-1]   #返回3
```

- 索引本质是复制。
- 左至右下标从0开始往后推。
- 右至左下标从-1开始往前推。

##### 切片：按下标切片

```python
list=['opaque','super',1,3]

usage：
    list[start:end:step]
    
example:
    list[0:2]     #返回['opaque''super']
    list[:]       #返回整个列表
    list[1:]      #返回下标为1到最后的元素列表
    list[-3:-1]   #返回[1,3]
```

- 步长默认为1
- 不指明start表示从下标0开始
- 不指明end表示到最后
- 切片到end但不包含end
- 切片的本质是复制
- 切片也是浅拷贝

##### index() #查元素对应下标

```python
list=['opaque','super',1,3]
usage:
    list.index(obj)
example:
    list.index('opaque')   #返回下标0
    list.index(1)          #返回下标2
    list[list.index(3)]    #返回元素3
```

##### count() #查元素出现的次数

```python
list=[1,2,1,1,4]

list.count(1)      #返回元素1在列表中出现的次数
```

#### **增相关**

##### append（） #列表最后追加一个元素

```python
list.append('nick')    #追加'nick'字符串
```

##### insert（） #在指定下标处插入一个元素

```python
list.insert(2,'sam')   #在下标为2的位置插入'sam'元素
```

- 原来该下标的元素向后推移一个下标
- 当要插入的下标大于当前列表长度时，就只是追加在后面，跟append（）一样效果

##### extend()、+  #列表合并

```python
list1=[1,2]
list2=[3,4]
list3=[5,6]
list4=[7,8]

list1+list2           #合并列表，得到[1,2,3,4]
list3.extend(list4)   #list4元素追加在list3之后，得到[5,6,7,8]
```

- extent()、append()、+的区别在于：前两者是原处修改列表，而+操作是用原列表的复制进行操作。因此原处修改的会更快。

#### **改相关**

##### 索引/分片赋值

```python
list=[1,2,3,4]

list[1]=5         #修改下标为1的元素的值
list[1:3]=[5,6]   #切片赋值修改
```

- 索引赋值和分片赋值都是原处修改。
- 分片赋值等号左右两边的元素可不等，即可以切两个元素但只赋值一个元素或任意个元素。
- 分片赋值可理解为先删除切片元素，然后执行插入元素操作。

#### **删相关**

##### remove() #删除指定元素

```python
list=[1,2,3,4,1]

list.remove(1)    #删除第一个元素1
```

- 删除第一个出现的，而不是全部

##### pop() #删除指定下标的元素

```python
list=[1,2,3,4,1]

list.pop(2)      #3会被删除且返回3
```

- 删除并返回被删除的元素
- 不指定元素时，会删除最后一个。

##### clear() #清空列表

```python
list=[1,2,3,4]

list.clear()     #列表被清空
```

##### del #删除索引出来的元素

```python
list=[1,2,3,4]

del list[0]       #下标为0的元素1被删除
del list[1:3]     #删除切片元素
del list          #删除变量list，整个列表将被删除
```

- del是全局的删除，即不是列表的独有方法，可以用于很多对象。

#### **其他方法**

##### reverse() #反转列表元素

```python
list=[1,2,3]

list.reverse()     #列表变为[3,2,1]
```

##### sort() #永久进行排序

```python
list.sort()
```

- 排序的规则按照ASCII码数值小到大。
- 永久排序是因为sort()在原处对列表进行修改。
- 一般原处修改操作的都不会有返回值（NONE）

##### copy() #拷贝列表

```python
list1=[1,2,[3,4]]
list2=list1.copy()
```

- 列表的copy()方法是浅copy，只能copy一层，即嵌套的还是共享引用。
- 想要迭代copy需要导入copy模块，然后使用copy.deepcopy()方法。

#### **一些理解**

- 列表名变量是指向列表对象的引用，列表又是0个或多个对象的引用。列表的浅拷贝实际上列表名变量成为了不同列表对象引用，但他们的列表对象都是指向相同元素对象的引用。

```python
list1=[1,2,3]
list2=list1.copy()
list3=list1

id(list1) == id(list2)   #False
list1 is list2           #False

id(list1) == id(list3)   #True
list1 is list3           #True

list1[0] is list2[0]     #True
list1[1] is list2[1]     #True
     .......

```

- 拷贝让list1和list2成为了不同列表对象的引用，但他们的列表对象都引用着相同的元素对象。修改任意一个列表的元素，实际上只是修改了列表对某个对象的引用而已，因此不会影响到另一个列表(因为两个列表是不同的对象)。
- list3=list1赋值操作，让list3和list1引用了同一个列表对象，因此更改这个列表对象引用的对象，list3与list1都会同时改变。

#### **二次总结**

```python
1.列表是可变的序列。
2.列表是任意对象的有序集合。
3.列表中的元素实质是列表对象引用了一个或多个其他对象。

4.强制类型转换：list()  #操作对象为序列
5.列表操作
	改：
		索引赋值
		  分片赋值：实质是先删除切片内容，再插入赋值的新内容。因此切片赋值不要求左右两边项相等。
	
	查：
		index()    #返回元素对应的下标
		count()    #返回元素在列表中的次数
	增：
		append()   #在最后追加一个元素（原处修改）。
		extend()   #将一个列表追加到另一个列表后面（原处修改）。
		insert()   #指定下标插入指定元素（原处修改）。
		   +       #序列合并（产生新对象）。
	
 	删：	
		remove()        #从列表中删除指定元素（原处修改）。
		pop()           #删除并弹出指定下标的函数（原处修改），不指定下标则删除并弹出最后一个元素。
		clear()         #清空列表（原处修改）。
		del list[0]     #删除索引出来的元素（不是列表特有的，也是原处修改）。
  
	其他：
		reverse()   #翻转列表（原处修改）
		sort()      #排序列表（原处修改）
		copy()      #浅拷贝


6.sort()与sorted()
	sort()是列表专有的，sorted()是内置函数，对于所有序列都可用
	sort()是原处修改，sorted()是产生新对象
	reverse()和reversed()也是一样的。
```





