---
title: Amazing Command -Windows篇
date: 2019-03-22 14:23:54
tags:
	- Winodows
categories: Misc
---

### 0x00 前面的话

记录一些自己觉得有用或有趣的DOS命令的用法，持续更新......

直接记录命令的某种用法，具体用法使用命令的`/?`的选项查看。（Window命令的`/？`选项就类似于Linux命令的`-h`）

<!-- more -->

### 0x01 DOS命令

#### **FOR**

> 格式：`FOR %variable IN (set) DO command [command-parameters]`
>
> > - `%variable`：指定一个单一字母可替换的参数。
> > - `(set)`：指定一个或一组文件。可以使用通配符。
> > - `command`：指定对每个文件执行的命令。
> > - `command-parameters`：为特定命令指定参数或命令行开关。

**用法一**：从`/R`指定的路径中找到符合set内特征的文件，并打印文件名。

> 格式：`FOR /R [[drive:]path] %variable IN (set) DO command [command-parameters]`

```c
FOR /R T:\PythonCode %i IN (*.py) DO @echo %i   //打印出T:\PythonCode下的所有后缀为.py的文件名。
FOR /R T:\PythonCode %i IN (.) DO @echo %i      //遍历T:\PythonCode下的所有目录，并打印目录路径。
FOR /R T:\PythonCode %i IN (*) DO @echo %i      //遍历T:\PythonCode下的所有目录文件，并打印文件名。
FOR /R %i IN (.) DO @echo %i                    //遍历当前目录下的目录树。
FOR /R %i IN (*) DO @echo %i                    //遍历当前目录下的文件名。
```

**用法二**：使用`/L`参数是，后面的set将变为一个产生数字迭代器。

> 格式：`FOR /L %variable IN (start,step,end) DO command [command-parameters]`

```c
FOR /L %i IN (1,1,254) DO @ping 192.168.100.%i -n 1 -w 1 | findstr /i ttl  

//ping 192.168.100.0这个网段，并筛选出ping通的主机
// -n 1表示只发一个echo-request。
// -w 1表示只等待echo-reply的时间，超时则认为目标不存活。
```

#### **findstr**

> 将其作为Window下的Grep使用。
>
> 常用参数：   
>
> - `/I`：大小写不敏感，默认敏感。 
> - `/M `：如果文件含有匹配项，只打印其文件名，而不是匹配字符所在的行。
> - `/N`：打印匹配字符所在的行号。
> - `/S`：在当前目录和所有子目录中搜索匹配文件。
> - `/F:filename`：从指定的文件中读取文件名列表作为要查找的对象。
> - `/G:file`：从指定的文件获得搜索字符串。

找到并打印出文件中的字符串行、筛选出字符串行。

```c
findstr "echo header" fish.php       //找到并打印fish.php文件中，包含'echo'或'header'字符串的行。
findstr /C:"echo header" fish.php    //找到并打印fish.php文件中，包含'echo header'这个字符串的行。
findstr /IMS "echo header"  *.php    //在当前目录及所有子目录的所有.php后缀文件中搜索字符串，只打印文件名。
```

配合`FOR`使用:

```c
FOR /R %i IN (*) DO @echo %i >> filelist.txt && findstr /IM /F:filelist.txt "echo"

// FOR语句会遍历当前目录下的所有文件并保存到filelist.txt
// findstr会使用'echo'字符串到filelist.txt文件中的所有文件去查找，若文件中有'echo'就打印文件名。

FOR /R D:\WWW\ %i IN (*) DO @echo %i >> filelist.txt && findstr /IM /F:filelist.txt /G:strlist.txt

//strlist.txt是包含许多一句话木马字符串的文件，就可以用来简单排查后门。
```

用于管道符之后进行结果筛选：

```c
netstat -ano | findstr /i est   //筛选出所有ESTABLISHED状态的链接。
netstat -ano | find "PID"       //找到指定进程号的连接。
```





