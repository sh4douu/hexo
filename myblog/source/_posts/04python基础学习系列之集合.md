---
title: Python基础学习系列Chapter 5：集合
date: 2019-03-19 16:27:06
tags: 
	- Python基础
	- Language
categories: 语言
---

### 0x00 集合概述

- 不可变对象的无序集合
- 存储的对象是唯一的，具有抗重性
- 属于可变对象类型
- 集合的操作都是产生的新的对象。所以intersection_update()、difference_update()、symmetric_difference_update()操作实际都是重新赋值。
- 集合类似无值的字典

<!-- more -->

### 0x01 集合创建

```python
set1={1,2,3,'a'}
set1=set([1,2,3,'a']) 
```

- 只能储存不可变对象
- 向set()传入一个序列或可迭代对象创建

### 0x02 集合操作

#### **交集 &**

##### intersection()

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5,7}

set1.intersection(set2)    #{1,3,5}
set1 & set2                #{1,3,5}
```

##### intersection_update() #用交集更新

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5,7}

set1.intersection_update(set2)    #set1变为{1，3，5}
```

##### isdisjoint() #无交集True，有交集False

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5,7}

set1.isdisjoint(set2)    #False
```

#### **并集 |** 

##### union()

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5,7}

set1.union(set2)   #{1,2,3,4,5,7}
set1 | set2        #{1,2,3,4,5,7}
```

- 集合是不可变的，操作不会影响set1，set2

#### **差集 -**

##### difference()

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5,7}

set1.difference(set2)   #set1有，set2无，{2,4}
set1 - set2             #set1有，set2无，{2,4}

set2.difference(set1)   #set2有，set1无，{7}
set2 - set1             #set2有，set1无，{7}
```

##### difference_update() #取差集并更新

#### **对称差集 ^**

##### symmetric_difference() #不包含交集后的并集

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5,7}

set1.symmetric_difference(set2)  #{2,4,7}
set1 ^ set2                      #{2,4,7}
```

##### symmetric_difference_update()

#### **父集、子集**

##### issuperset()、issubset()

```python
set1 = {1,2,3,4,5}
set2 = {1,3,5}

set1.issuperset(set2)   #True
set2.issubset(set1)     #True
```

#### **增相关**

##### add() #增加一个

```python
set1 = {1,3,5}

set1.add(7)
```

##### updata() #合并

```python
set1={1,2}
set2={3,4}

set1.update()
```

- 原处改变

#### **删相关**

##### remove() #删除一个元素

```python
set1={1,2,3}

set1.remove(2)
```

##### discard() #删除一个元素

```python
set1={1,2,3}

set1.discard(2)
```

- remove与discard区别在于，remove删除一个不存在元素会报错，而discard不会。

##### pop() #随机删除一个元素并返回该元素