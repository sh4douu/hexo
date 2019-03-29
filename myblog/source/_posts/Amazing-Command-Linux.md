---
title: Amazing Command -Linux篇
date: 2019-03-22 14:24:08
tags: 
	- Linux
categories: Misc
---

### 前面的话

记录一些自己觉得有用或有趣的Linux命令的用法，持续更新......

直接记录命令的某种用法，具体用法使用命令的`-h`的选项查看。

<!-- more -->

### 0x00 awk

> awk默认的行为是从标准输入中逐行（即每次读取直到遇到换行符`\n`停止）读入数据，并以空格为默认分隔符将读取的每行数据切片，`$0`指代整行数据，`$1`指代切片得到的第一部分，`$2`指代切片得到的第二部分，以此类推,`$n`指代切片得到的第n部分。

**awk + action**：对awk的默认行为得到的数据直接进行action。

- 格式：`awk '{action}'`
- last -n 2 | awk '{print $0}' 

```
nick     :0           :0               Fri Mar 29 10:08   still logged in
reboot   system boot  4.18.0-16-generi Fri Mar 29 10:07   still running
```

- last -n 2 | awk '{print $1}' 

```
nick  
reboot  
```

**awk + pattern + action**：对读取的每行先使用pattern正则去匹配，满足正则的才进行awk的默认行为，然后进行action。

- 格式：`awk '/pattern/{action}'`
- last -n 4 | awk '/nick/{print $0}'

```
nick     :0           :0               Fri Mar 29 10:08   still logged in
nick     :0           :0               Thu Mar 28 10:48 - 17:16  (06:27)
```

- last -n 4 | awk '/^nick/'

```
reboot   system boot  4.18.0-16-generi Fri Mar 29 10:07   still running
reboot   system boot  4.18.0-16-generi Thu Mar 28 10:47 - 17:16  (06:28)
```

`-F` 用于指定分割符，默认是空格。printf()用来格式化输出。

- cat /etc/passwd | awk -F ':' '/root/{printf("%s,%s",$1,$7)}'

```
root,/bin/zsh% 
```

### 0x01 screenfetch

> 用于显示系统基本信息及其banner。

- 直接`screenfetch`会显示系统基本信息及其对应的ASCII字符banner。
- `screenfetch -A 'OS_name'`：显示系统基本信息及指定系统的ASCII字符banner。

```
#用于你可能是ubuntu，但是却希望打印的图标是Mac或其他
screenfetch -A 'Mac OS X'
screenfetch -A 'Kali Linux'
```

- `screenfetch -s`：屏幕快照，即截屏。

