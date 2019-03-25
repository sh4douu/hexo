---
title: Python基础学习系列Chapter 2：字符串
date: 2019-03-18 16:50:44
tags:
	- Python基础
	- Language
categories: 语言
---

### 0x00 字符串概述

字符串是不可变的序列。

Python3.0字符串分3中类型：str用于Unicode文本；bytes用于二进制文本；bytearray是bytes的变体。

### 0x01 字符串

#### 1.字符串常量：

```python
'string'
"string"
'''string'''
"""string"""
r'string'     #Raw字符串
b'string'     #bytes字符串
```

<!-- more -->

#### 2.字符串操作

- 索引：`str[下标]`
- 切片：`str[下标开始:下标结束：步长]`    #步长默认为1
- 合并：`+`、`*`
- 类型强制转换：`str()`
- 修改字符串：可以通过产生新对象后重新赋值给原变量。

#### 3.字符串格式化：

```python
"this is %s" %'collision'               #字符串形式
"%s is %s boy"%("collision","good")     #元组形式
"%(x)d,%(y)d"%{"x":1,"y":2}             #字典形式
```

format格式化

```python
'a{}c{}'.format('b','d')
'a{0}c{1}'.format('b','d')
'a{name}c{age}'.format(name='b',age='d')
'a{name}c{age}'.format_map({'name':'b','age'='d'})
```

### 0x02 字符串方法：	

```python
查找：
	count()          #返回字符或字符串在字符串中出现的次数
	find()、rfind()  #查找字符或字符串返回第一次出现时的下标
	index()          #返回字符或字符串在字符串中的下标	
	
	PS:find()与index()区别在于，find对不存在的字符串查找会返回-1，而index会报错。

判断：
	startswith()、endswith()     #判断字符或字符串的开始与结尾
	
	isalpha()        #判断字符串是否是字母组成	
	isnumeric()      #字符串只包含数字字符	
	isalnum()        #判断字符串是否是字母或数字组
	
	isdecimal()      #判断字符串是否是十进制数字
	isdigit()        #判断字符串是否是数字组成
	islower()        #字母字符是否都是小写字母
	isupper()        #都是大写
		
操作：
	join()     #将字符串作为分隔符，合并一个序列为一个字符串。	
		PS：序列的元素要求是字符串才可以。
		
	split()    #以指定字符串为分隔符分割一个字符串为一个列表	
	replace()  #对应替换								
	strip()    #脱掉字符串两端的指定字符串（默认脱空格）		
	
	lower()    #将字符串中的字母字符变为小写
	upper()    #变大写

编码：
	'abc'.encode('utf-8')	
```

#### **字符串方法**

#### C

##### capitalize() #将字符串首字符大写

```python
'abc'.capitalize()   #返回'Abc'
```

- 若首字符是字母之外的则不做改变。

##### center() #居中字符串并填充

```python
'abc'.center(50)      #居中字符串，不足50个字符用空格填充
'abc'.center(50,'-')  #字符串不足50个字符时，用'-'在两边进行填充
```

- 数量为奇数时，优先填充右边。
- ljust()为左对齐填充。
- rjust()为右对齐填充。

##### count() #返回字符或字符串在字符串中出现的次数

```python
'abcac'.count('a')     #返回2
'abcacac'.count('ac')  #返回2
```

#### E

##### encode() #对字符串进行编码

```python
'abc'.encode('utf-8')   #等同于b'abc',因为默认编码utf-8
'abc'.encode('gb2312')
```

- 编码即将字符串以特定编码方式转为二进制。

##### startwith()、endwith() #判断字符串开始与结尾

```python
'abc'.startwith('ab')  #返回True
'abc'.endwith('bc')    #返回True
```

#### F

##### find()、rfind() #查找字符串返回下标

```python
'abcd'.find('bc')    #返回2
'abccd'.rfind('c')   #返回最右边c的下标3
```

- rfind()返回的是最右边匹配的字符串的下标。
- 查找的字符串不存在时，返回-1

##### format()、format_map()#字符串格式化

```python
'a{}c{}'.format('b','d')
'a{name}c{age}'.format(name='b',age='d')
'a{name}c{age}'.format_map({'name':'b','age':'d'})
```

#### I

##### index() #返回字符或字符串在字符串中的下标

```python
'abc'.index('b')    #返回1
```

- 与find()区别在于，不存在字符串时会报错

##### isalnum() #判断字符串是否是字母或数字组成

```python
'abc123'.isalnum()  #True
'abc'.isalnum()     #True
'123'.isalnum()     #True
'abc\n1'.isalnum()  #False
'abc1.1'.isalnum()  #False
```

##### isalpha() #判断字符串是否是字母组成

```python
'aBc'.isalpha()    #True
'aB2'.isalpha()    #False
```

##### isdecimal() #判断字符串是否是十进制数字

```python
'12'.isdecimal()   #True
```

##### isdigit() #判断字符串是否是数字组成

```python
'123'.digit()    #True
'a1x2'.digit()   #False
```

##### islower() #字母字符是否都是小写字母

```python
'abc123'.islower()   #True
'Aabc'.islower()     #False
```

- 字符串可以存在其他字符，只要字母字符是小写即可True.
- isupper()与之相反，不在赘述

##### isidentifier()#判断字符串可否符合变量名命名规则

##### isnumeric() #字符串只包含数字字符

##### isspace()#字符串只包含空格

##### istitle()#字符串是否是标题化的

```python
'This Is Op'.istitle()   #True
'this Is Op'.istitle()   #False
```

- title()用于将字符串标题化，即单词首字母大写

#### J

##### join()  #将一个序列以分隔符分割合并为一个字符串

```python
''.join(('a','b','c'))   #'abc'
'_'.join(['a','b','c'])  #'a_b_c'
```

- 即以字符串为分隔符，将一个序列合并为一个字符串

##### split() #以指定字符串为分隔符分割一个字符串为一个列表

```python
'abc'.split('b')       #['a','c']
'a_b_c'.split('_')     #['a','b','c']
'a_b_c'.split('_',1)   #['a','b_c']
```

- 第一个参数是分隔符，第二个参数是分割的次数
- splitlines() #默认是以\n、\r、\r\n作为分隔符
- rsplit()表示从右边开始进行分割

#### L

##### lower() #将字符串中的字母字符变为小写

##### upper() #变大写

#### M

##### maketrans() #设置转换规则

##### translate() #根据规则进行转换

```python
res=str.maketrans('abcd','1234')    #设置规则
'abcd'.translate(res)     #'1234'
'abcA'.translate(res)     #'123A'
```

#### R

##### replace() #对应替换

```python
'abc'.replace('b','B')    #'aBc'
```

- 本质是产生了一个新的字符串，因为字符串时不可变对象

#### S

##### strip() #脱掉字符串两端的指定字符串

```python
'abc\n'.strip()      #'abc'
'abca'.strip('a')    #'bc'

a='abc'
a.strip('c')   #'ab'
print(a)       #'abc'
```

- 不指定字符串默认脱掉换行符、制表符等
- 脱掉并返回脱掉后剩下的字符串
- ==该操作不会对原字符串产生影响，因为字符串是不可变的。该操作只是产生一个新的字符串，原字符串仍然未变，效果就是字符串只是暂时脱掉而已==
- lstrip()表示只脱掉最左端的
- rstrip()表示只脱最右边的

##### swapcase()#大小写翻转，大写变小写，小变大

##### zfill()#右对齐，不足填0

```python
'abc'.zfill(5)    #'00abc'
```





