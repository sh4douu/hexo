---
title: XML基本知识学习
date: 2019-02-28 15:10:47
tags: 
	- XML
	- Language
categories:
	- 语言
---

## 0x00 XML概述

- XML 指可扩展标记语言（eXtensible Markup Language）。
- XML 的设计宗旨是传输数据，而不是显示数据。
- XML 标签没有被预定义。
- XML 被设计用来结构化、存储以及传输信息，实际上XML没有做任何事情。

> XML只是以某种结构化形式组织数据，说它没有做任何事是他不会像HTML被渲染为形象色色的网页。

<!-- more -->

**XML与HTML**

1.XML 和 HTML 为不同的目的而设计：

> - XML 被设计用来传输和存储数据。
> - HTML 被设计用来显示数据。

2.HTML 中使用的标签都是预定义的，XML 允许创作者定义自己的标签和自己的文档结构。

**3.XML 是独立于软件和硬件的信息传输工具。**

## 0x01 XML结构

> XML文档是树形结构的，由根元素扩展，且必须包含根元素。

**一个栗子:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<note date="9102/2/28">
  <to>Mikasa</to>
  <from>Allen</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
```

- 第一行是XML声明，指定XML版本以及编码方式，声明是可选的。
- `<note>`为根元素，XML标签都是成对且闭合的，以`</note>`闭合标签。
- XML是大小写敏感的，`<note>`与`<Note>`是不同的标签。
- 标签是可以设置属性的，如例子中`<note>`标签的`date`，但是属性的值必须使用引号括起来。

**实体引用：**

在 XML 中，一些字符拥有特殊的意义。

例如，如果把字符 "<" 放在 XML 元素中，会发生错误，这是因为解析器会把它当作新元素的开始。

```xml
<!-- 使用下面会发生错误 -->
<message>if salary < 1000 then</message>
```

为了避免这个错误，用**实体引用**来代替 "<" 字符：

```xml
<message>if salary &lt; 1000 then</message>
```

XML有5个预定义的实体引用：

![](xml\QQ截图20190228152213.png)

## 0x02 DTD

文档类型定义（DTD）可定义合法的XML文档构建模块。

**1.为什么使用DTD？**

通过 DTD，您的每一个 XML 文件均可携带一个有关其自身格式的描述。

通过 DTD，独立的团体可一致地使用某个标准的 DTD 来交换数据。

还可以用DTD 来验证从外部接收到的、自身的数据。

**2.DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。**

i.在文档内声明的栗子：`<!DOCTYPE root-element [element-declarations]>`

```xml
<?xml version="1.0"?>
<!DOCTYPE note [    #定义此文档是 note 类型的文档。
  <!ELEMENT note (to,from,heading,body)>   #定义note元素有四个元素："to、from、heading、body"
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT from (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body (#PCDATA)>   #定义body元素为"#PCDATA"类型
]>
<note>
  <to>Mikasa</to>
  <from>Allen</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend</body>
</note>
```

ii.作为外部引用的例子：`<!DOCTYPE root-element SYSTEM "filename">`

```xml
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">  #引用外部文件note.dtd
<note>
  <to>Mikasa</to>
  <from>Allen</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
```

`note.dtd`文件内容：

```xml-dtd
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
```

**3.DTD实体**

**实体**是用于定义引用普通文本或特殊字符的快捷方式的变量。

**实体引用**是对实体的引用。引用后会获得实体指向的实际数据。

**实体声明也是在DTD声明中，与元素声明同级。**

i.在文档内部声明实体的例子：`<!ENTITY entity-name "entity-value">`

```xml
<?xml version="1.0"?>
<!DOCTYPE note [    
  <!ELEMENT note ANY>   #定义note元素可以有任意元素。
  
  <!ENTITY entity1 "Be">      #文档内声明
  <!ENTITY entity2 "back.">
]>
<note>&entity1; &entity2;</note>
```

ii.外部声明引用实体的例子：`<!ENTITY entity-name SYSTEM "URI/URL">` or `<!DOCTYPE 根元素 PUBLIC "public_ID" "文件名">`

```xml
<?xml version="1.0"?>
<!DOCTYPE note [    
  <!ELEMENT note ANY>   
  
  <!ENTITY entity1 SYSTEM "http://www.sakuxa.com/example.xml">  #system关键字表示外部引用
]>
<note> &entity1;</note>
```

iii.参数实体

参数实体只用于 DTD 和文档的内部子集中。只有在DTD中才能引用参数实体，参数实体的声明和引用都是以百分号%。并且参数实体的引用在DTD是理解解析的，替换文本将变成DTD的一部分。

**记住：%entity; 会被它指向的数据进行直接替换。**

```xml
<?xml version="1.0"?>
<!DOCTYPE note [    
  <!ELEMENT note ANY>   
  
  <!ENTITY % entity1 SYSTEM "http://www.sakuxa.com/example.dtd">
  %entity1  #会被example.dtd内容替换
]>
<note> &entity2;</note>
```

`example.dtd`文件内容：

```xml-dtd
<!ENTITY entity2 "Be back.">
```

**注意：** 一个实体由三部分构成: 一个和号 (&), 一个实体名称, 以及一个分号 (;)。