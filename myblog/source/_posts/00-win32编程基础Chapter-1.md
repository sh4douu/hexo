---
title: Win32编程基础Chapter 1：80386 CPU
date: 2019-04-01 09:37:37
tags: 
	- Win32
	- 汇编语言
categories: Win32
---

#### 0x00 80386处理器有三种工作模式

- 实模式：为兼容8086处理器的模式，该模式下80386与8086工作等同。DOS操作系统运行于该模式下。
- 保护模式：32位寻址，寻址空间4GB，支持内存分页，多任务等。Windows操作系统运行于该模式下。
- 虚拟86模式：为了在保护模式下运行8086程序而产生的模式。

<!-- more -->

#### 0x01 优先级机制

- 在保护模式下，80386CPU还支持优先级机制，不同的程序可以运行在不同的优先级上。
- 分四个优先级（0-4）,操作系统运行在0优先级上，应用程序运行在比较低的优先级上。

#### 0x02 寻址

- 在80386处理器中，有32根地址总线，寻址空间为4G，而通用寄存器也都是32位的，意味着不再需要段地址+偏移地址的方式进行寻址，可直接使用任意通用寄存器进行寻址。
- 虽然不再需要段地址+偏移地址的方式进行寻址，但是并不意味着段寄存器就没用了，且看下面。

#### 0x03 段描述符

在保护模式下，一个地址空间能否被写入、可以被多少优先级的代码写入以及是否允许被执行等安全问题需要被考虑，因此就需要为一个地址空间定义一些安全方面的属性。

这些属性需要用64位长的数据才能表示完全，这64位长的数据称为段描述符（Segment Descriptor）。

段描述符包含一个地址空间（段）的：基址（起始地址）、限长、优先级等。这个空间就相当于一个段。一个个地址空间相当于一个个段。

#### 0x04 段描述符表与段选择器

由于段寄存器是16位的，并不能存放64位长的段描述符，解决办法是将所有地址空间的段描述符在指定内存中顺序存放，从而形成一个段描述符表（Descriptor Table）。

而16位段寄存器用来从段描述符表中索引一个地址空间的段描述符。此时，段寄存器不再存放段地址，而是作为段选择器（Segment Selector）。

#### 0x05 段描述符表分为两种：

GDT：Global Descriptor Table,全局描述符表。

> 它包含系统中所有任务都可用的段描述符，通常包含操作系统使用的代码段、数据段、栈段的描述符以及各个任务的LDT段等。全局描述符表只有一个。即主要是一些公用的资源的描述符。

LDT：Local Descriptor Table,局部描述符表。

> 每个任务都有自己独立的LDT，它包含任务的数据段、代码段、栈段等的描述符。即任务内部的内存也根据属性不同划分为不同的内存段。

PS：每个任务的局部描述符表使用不同的内存段存储，存储局部描述符表所占用的内存段的段描述符被当作系统描述符放在GDT中。

#### 0x06 80386处理器引入两个新的寄存器用来管理段描述符表：

GDTR：是一个48位寄存器，指向全局描述符表（GDT），高32位存储GDT的基地址，是直接的内存地址。低16位储存限长（表的长度）。

LDTR：是一个16位寄存器，与GDTR不同，LDTR与其他段寄存器一样，存储的是索引值。存储的是局部描述表内存段的段描述符在GDT中的索引。

#### 0x07 16位的段寄存器（CS、DS、ES、FS、GS、SS）：

实际只用高13位来作为索引值。

剩下三个比特位：

- 第0、1位：程序的优先级位RPL。
- 第3位： TI位，用于决定在哪个描述符表中进行索引：
  - TI=0，在GDT中索引段描述符。
  - TI=1，在LDT中索引段描述符。

#### 0x08 总结

GDT主要存放一些公用的资源以及各个任务的LDT内存段的描述符。GDT只有一个

LDT主要存放各个任务的栈段、数据段等内存段的描述符，LDT可以有多个，LDT本身所占用的内存段的描述符存储在GDT中。

GDTR是48位的寄存器，存储（指向）GDT的内存的起始地址。

LDTR是16位的寄存器，作为到GDT中寻找任务的LDT内存段的索引。

> 要找到任务中各个段的描述符，首先要找到任务的LDT，要找到任务的LDT，就使用LDTR到GDT中去索引查找。

16位的寄存器是作为最终在GDT或LDT中索引的值。至于在哪个表索引，取决于第3个比特位。

![](00-win32编程基础Chapter-1\QQ截图20190401094016.png)

#### 0x09 分页机制

经过以上操作得到的地址称为线性地址：

- 若未启用分页机制，得到的线性地址就是物理内存地址
- 若启用了分页机制，得到的线性地址是一个虚拟地址，通过页表映射的真正的物理内存地址。 

分页机制解决了8086的内存碎片问题。

页表不仅规定了地址的映射，还规定了页表的属性，如，是否可读、可写、可执行。

**Windows是分时多任务操作系统，CPU时间被划分为一个个时间片后分配给不同程序，然后不断切换使用。即CPU在某一时间片内实际只运行一个程序，那么在该时间片内，除了当前时间片的程序外其他程序并不需要被加载到内存中，这也是为什么每个程序都可以使用2\*\*32B的内存空间大小的原因。**

> 这也是得益于分页机制存在的原因，因为分页机制会自动将线性地址映射为内存的物理地址，因此我们只需要针对这4GB内存空间编程即可，至于如何将我们程序的地址转换为物理地址便交给操作系统的分页机制。

操作系统DLL由于要被所有程序调用，所以会一直存在内存中；另外一些用户DLL可能被多个程序使用，那么在这些程序的时间片内，也会一直存在于内存中。

#### 0x0a Win32 寄存器

32位通用寄存器：EAX、EBX、ECX、EDX,它们的低16位对应：AX、BX、CX、DX,它们的低16位又可分为高8位和低8位：AH\AL 、BH\BL 、CH\CL 、 DH\DL。

有2个32位寄存器ESI和EDI称为变址寄存器。其低16位对应先前CPU中的SI和DI。它们主要用于存放存储单元在段内的偏移量，用它们可实现多种存储器操作数的寻址方式。

有2个32位寄存器EBP和ESP。其低16位对应先前CPU中的BP和SP。寄存器EBP、ESP、BP和SP称为指针寄存器。

- BP 为基指针(Base Pointer)寄存器，用它可直接存取堆栈中的数据。
- SP为堆栈指针(Stack Pointer)寄存器，用它只可访问栈顶。

32位CPU把指令指针扩展到32位，并记作EIP，EIP的低16位与先前CPU中的IP作用相同。

段寄存器都不再被需要给出段地址，变成了段选择器：CS、DS、SS、ES、FS、GS。在Win32编程中，用户基本不用关心段寄存器，因为它们是操作系统自动安排好的。