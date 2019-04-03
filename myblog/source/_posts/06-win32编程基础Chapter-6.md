---
title: Win32编程基础Chapter 6：第一个窗口程序
date: 2019-04-02 09:26:59
tags:
	- Win32
	- 汇编语言
categories: Win32
---

#### 0x00 程序与窗口的关系 

一个窗口不一定就是一个程序，它可能只是程序的一部分，一个程序可以有多个顶层窗口，一个顶层窗口又可以有多个子窗口，子窗口又可以有子窗口。

程序也不一定是窗口，如果一个程序不需要与用户进行交互，那么他可以不建立窗口。

DOS程序是过程驱动的。窗口程序是事件驱动的。

<!-- more -->

![](06-win32编程基础Chapter-6\Image.png)

#### 0x01 消息队列

Windows在系统内部有一个系统队列，当输入设备有所动作（键盘输入、鼠标移动/点击等），Winodws会产生相应的记录并放到系统消息队列中。同时，Windows还会为每个窗口程序（严格说是每个线程）维护一个消息队列。

![](06-win32编程基础Chapter-6\Image [2].png)

#### 0x02 模块与句柄

模块：

> 一个运行中的.exe或dll就是一个模块，包括代码和所有资源。即在硬盘上的文件不能称为模块，加载到内存后才能称为模块。dll被不同程序加载到内存就产生了不同模块，以隔离地址空间。

句柄：

> 为了能够区分不同模块，因此产生了句柄用来唯一标识每个模块。句柄只是一个数值，它对程序是没意义的，它只是Windows用来区分不同资源的编号，以便Window能够区分和使用。

简单来说：

> Windows为每个窗口、dll等都各分配一个句柄，并记录窗口、dll等与句柄的对应关系。
>
> Windows将句柄给程序，程序操作句柄等同于操作窗口。
>
> Windows也可以通过我们传入的句柄知道程序要操作的窗口。
>
> Windows上几乎所有东西都用句柄标识：文件句柄、窗口句柄、模块句柄、线程句柄等。

获取句柄：

> 在Win32中，模块句柄数值上等于程序装入内存的起始地址。

可以用API函数GetModuleHandle获取模块的句柄：

```assembly
invoke GetModuleHandle,lpModuleName  ;lpModuleName参数是指向模块名称字符串地址的指针。
```

PS：如果lpModuleName参数使用NULL，返回得到的是调用者自身的本模块句柄。为了便利会先把程序的句柄获取并放在全局变量保存。

API函数返回值即模块的句柄，存储在EAX中。 

  

#### 0x03 窗口程序

1.一个程序的结构流程：

> 得到应用程序的句柄（GetModuleHandle）
>
> 注册窗口类(RegisterClassEx) 
>
> 建立窗口（CreateWindowEx） 
>
> 显示窗口（ShowWindows） 
>
> 刷新窗口客户区（UpdateWindow） 
>
> 进入无限的消息获取和处理的循环：
>
> - 获得消息（GetMessage）
> - 消息处理（TranslateMessage ）
> - 分派消息（DispatchMessage）

2.注册窗口类：

> 建立窗口类的方法是在系统中注册，注册窗口类的API函数是 `RegisterClassEx`，最后的“Ex”是扩展的意思，因为它是 Win16 中`RegisterClass`的扩展。

一个窗口类定义了窗口的一些主要属性，如：图标、光标、背景色、菜单和负责处理所属该窗口消息的函数。这些属性并不是分成多个参数传递过去的（这种做法不聪明），而是定义在一个`WNDCLASSEX`结构中，再把结构的地址当参数一次性传递给 `RegisterClassEx`。

WNDCLASSEX结构定义：

```assembly
WNDCLASSEX STRUCT 

CbSize         DWORD   ?  ;结构的字节数
Style          DWORD   ?  ;类风格  
LpfnWndProc    DWORD   ?  **;窗口过程的地址,****回调函数的地址。**
CbClsExtra     DWORD   ?  
CbWndExtra     DWORD   ?  
HInstance      DWORD   ?  ;**指定窗口类属于的模块，传入模块的句柄。**  
HIcon          DWORD   ?  ;窗口左上角的图标  
HCursor        DWORD   ?  ;窗口光标  
HbrBackground  DWORD   ?  ;窗口客户区的背景色  
LpszMenuName   DWORD   ?  ;窗口菜单  
LpszClassName  DWORD   ?  **;要注册的类的字符串名字，指向类名字符串地址的指针。**  
HIconSm        DWORD   ?  ;小图标 

WNDCLASSEX ENDS
```

- LpfnWndProc

> 基于这个类创建的窗口都使用该回调函数，即一个窗口过程可以服务多个窗口，只要它们是由同一个窗口类创建的。
>
> Windows运行到DispatchMessage函数时，会把消息传给该过程函数。

注册窗口类：

```assembly
local   stWndClass:WNDCLASSEX             ;以WNDCLASSEX结构为类型定义一个结构。
invoke  RegisterClassEx,addr stWndClass   ;调用注册函数进行窗口类注册
```

3.建立窗口：

> 利用上一步注册的窗口类为基础（模板）来创建一个窗口。

和注册窗口类时用一个结构传递所有参数不同，建立窗口时所有的属性都是用单个参数的方式传递的，建立窗口的API函数是`CreateWindowEx`。  

建立窗口：

```assembly
invoke   CreateWindowEx, dwExStyle, lpClassName,lpWindowName, dwStyle, x, y, nWidth, nHeight, hWndParent, hMenu, hInstance,lpParam
```

参数解析：

- `lpClassName`：传入上一步注册的类的字符串名称。表示使用该类为模板创建窗口，会继承窗口类的属性以及使用窗口类指定的窗口过程。 
- `lpWindowName`：窗口的标题。指向窗口标题字符串地址的指针。 
- `hInstance`：窗口所属的模块的句柄。
- `dwExStyle、dwStyle`：窗口的两个参数 dwStyle 和 dwExStyle 决定了窗口的外形和行为，们是一些以WS（Windows Style的缩写）为开头的预定义值，具体查文档。
- `x, y`：指定窗口左上角位置，单位是像素（px）。即窗口的位置。
- `nWidth、nHeight`：窗口的宽度和高度，也就是窗口的大小，同样是以像素为单位的。

CreateWindowEx 建立窗口以后，eax 中传回来的是窗口句柄，注意要把它先保存起来，因为这时候，窗口虽已建立，但还没有在屏幕上显示出来。

4.显示窗口：

> `ShowWindow`用于显示上一步创建的窗口，该函数有两个参数，第一个参数是要显示的窗口的句柄，第二个参数是显示的方式，显示的方式是众多预定义的值，具体定义查文档。

5.绘制窗口：

> 窗口显示以后，用 `UpdateWindow` 绘制客户区，它实际上就是向窗口发送了一条 `WM_PAINT` 消息。到此为止，一个顶层窗口才算正常建立并显示！ 

绘制窗口，传入要绘制的窗口的句柄：

```assembly
invoke UpdateWindow,hxxxx
```

6.消息循环：

消息循环中的几个函数要用到一个MSG结构，用来做消息传递：

```assembly
MSG STRUCT  

Hwnd      DWORD     ?   ;消息要发向的窗口的句柄。
Message   DWORD     ?   ;消息标识符，在头文件中以WM_开头的预定义值。
WParam    DWORD     ?   ;消息的参数之一。
LParam    DWORD     ?   ;消息的参数之二。
Time      DWORD     ?   ;消息放入消息队列的时间。
Pt        POINT     <>  ;这是一个POINT数据结构，表示消息放入消息队列时的鼠标坐标。

MSG ENDS
```

获取消息：

> `GetMessage` 函数从消息队列里取得消息，用取得的消息填写到传入的MSG结构。

```assembly
invoke  GetMessage, lpMsg, hWnd, wMsgFilterMin,wMsgFilterMax ;
```

参数解析：

- lpMsg：指向一个MSG结构，函数取到的消息填写MSG后返回。
- hWnd：传入要获取消息的的窗口句柄。指定为NULL表示获取所有本程序的窗口的消息。
- wMsgFilterMin 和wMsgFilterMax 为0表示获取所有编号的消息。

`TranslateMessage`的作用是遇到键盘消息则将扫描码转换成ASCII码并在消息队列中插入`WM_CHAR` 或` WM_SYSCHAR `消息，参数就是转换好的ASCII码。   

最后，由 `DispatchMessage` 将消息（`GetMessage`填写好的MSG结构）发送到窗口对应的窗口过程去处理。

窗口过程返回后 `DispatchMessage` 函数才返回，然后开始新一轮消息循环。



#### 0x04 Win窗口程序编写模板

该模板是用C调用API函数编写的。

```c
#include <windows.h>

LRESULT CALLBACK WindowProc(HWND hwhd,UINT,WPARAM wParam,LPARAM IParam);   //回调函数（过程函数）原型。

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPreInstance,PSTR szCmdline,int iCmdShow )   //WinMain函数由系统调用。

{
     \* ------------------------注册窗口类---------------------------------*\

     static TCHAR szAppName[] = TEXT("MyWindows");     //声明并初始化一个字符串，用作注册的窗口类的名称
     
     WNDCLASS wndclass;                          //以WNDCLASS结构为模板，声明一个结构名为wndclass的结构。
     wndclass.style = CS_HREDRAW | CS_VREDRAW;
     wndclass.lpfnWndProc = WindowProc;          //指明回调函数名称（函数名本质是地址，因此是指针）。
     wndclass.cbClsExtra = 0;
     wndclass.cbWndExtra = 0;
     wndclass.hInstance = hInstance;             //程序当前实例的句柄，由WinMain函数传入。
     wndclass.hIcon = LoadIcon(NULL, IDI_APPLICATION);              //图标
     wndclass.hCursor = LoadCursor(NULL, IDC_ARROW);                //光标
     wndclass.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);  //窗口背景色
     wndclass.lpszMenuName = NULL;                                  //窗口菜单
     wndclass.lpszClassName = szAppName;     //注册的窗口类的字符串名称。（要求指针，因此是字符串的地址）

     RegisterClass(&wndclass);     //注册窗口类，类的信息定义在WNDCLASS结构中。传入结构的地址作为参数。

     
     \* ------------------------创建窗口---------------------------------*\

     HWND hwnd;     //声明一个句柄结构类型的变量。(通常声明于WinMain函数内部顶部，此处为了方便观看)

     //根据窗口类创建窗口，并返回窗口的句柄。
     hwnd = CreateWindow( szAppName,                  //上一步中注册的类的名称（字符串地址）。
                         TEXT("MyWindow"),           //窗口的标题（字符串地址）。
                         WS_OVERLAPPEDWINDOW,
                         CW_USEDEFAULT,
                         CW_USEDEFAULT,
                         CW_USEDEFAULT,
                         CW_USEDEFAULT,
                         NULL,
                         NULL,
                         hInstance,                  //程序当前实例的句柄，由WinMain函数传入。
                         NULL);


     \* ------------------------显示、绘制窗口---------------------------------*/

     ShowWindow(hwnd,iCmdShow);      //传入要显示的窗口的句柄。
     UpdateWindow(hwnd);             //传入要绘制的窗口句柄，实际上就是向窗口发送了一条 WM_PAINT 消息。

     \* ------------------------消息处理---------------------------------*/

     MSG msg;        //一个MSG结构为类型，声明一个结构名为msg的结构。
     
     while(GetMessage(&msg,NULL,0,0))    //传入的时msg结构的地址，因此三个函数操作的都是同一的对象。
     {
         TranslateMessage(&msg);
         DispatchMessage(&msg);          //将消息分发给回调函数WindowProc**

     }

     return msg.wParam;  //当接收到一个WM_QUIT消息时，程序就中止。并且返回传递给WM_QUIT消息的wParam参数的值。

}



LRESULT CALLBACK WindowProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)  //回调函数定义

{

    HDC hdc;
    PAINTSTRUCT ps;
    RECT rect;

    switch (message)
    {
       case WM_PAINT:
           hdc = BeginPaint(hwnd, &ps);
           GetClientRect(hwnd, &rect);
           DrawText(hdc, TEXT("Hello World!"), -1, &rect,DT_SINGLELINE | DT_CENTER | DT_VCENTER);
           EndPaint(hwnd, &ps);
           return 0;
            
       case WM_DESTROY:
           PostQuitMessage(0);
           return 0;
    }

    return DefWindowProc(hwnd, message, wParam, lParam); //若消息没有匹配到上面，默认执行该操作。

}
```



