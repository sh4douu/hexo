---
title: Python模块 Chapter 3：subprocess
date: 2019-04-11 20:25:26
tags:
	- Python模块
	- Language
categories: Python
---



### 0x00 概述

`subprocess`模块允许你创建一个新的进程，新建进程是当前程序的子进程。可以连接到新建进程的input/output/error pipes, 并且可以获取到它们的返回码。

这个模块是为了取代下面这些比较老的模块功能：

```python
os.system
os.spawn*
```

subprocess模块提供两种方法来创建新的进程：

- `subprocess.run()`
- `subprocess.Popen()`

> 使用`run()`方法创建新进程可以满足绝大多数需求，但`Popen（）`类提供比`run()`更高级更灵活的用法。

<!-- more -->

### 0x01 subprocess.Popen()

> 产生一个子进程执行程序：
>
> - 对于 POSIX 系统, 该类调用`os.execvp()`函数来创建子进程。
> - 对于 Windows 系统,该类调用Windows API函数 `CreateProcess()` 来创建子进程。

**Popen类的构造方法：**

```python
def __init__(self, args, bufsize=-1, executable=None,
                 stdin=None, stdout=None, stderr=None,
                 preexec_fn=None, close_fds=_PLATFORM_DEFAULT_CLOSE_FDS,
                 shell=False, cwd=None, env=None, universal_newlines=False,
                 startupinfo=None, creationflags=0,
                 restore_signals=True, start_new_session=False,
                 pass_fds=(), *, encoding=None, errors=None):
```

- **args**：该参数要求是一个序列类型（list、tuple...）或字符串（str）。

> 如果**args**是一个序列，那么序列的第一个元素被作为程序名（路径），之后的作为参数。
>
> 在Windows上时，如果**args**是一个序列，会被转换为字符串。

- **shell**：为`True`时，使用系统shell执行**args**指定的程序，默认是`False`。

> On POSIX with `shell=True`, the shell defaults to `/bin/sh`.
>
> 对于Windows，你只有在需要执行内置DOS命令时（例如，`dir`、`ipconfig`...），才使用`shell=True`。运行批处理文件（.bat）或可执行文件时没有必要将`shell`参数设为`True`。

- **stdin、stdout、stderr**： 指定执行程序的标准输入、标准输出、标准错误输出的文件句柄。

> 若不指定则默认都为`None`，表示不进行重定向，那么子进程会从父进程继承文件句柄。
>
> 若指定了，即进行重定向。有效值可以为`subprocess.PIPE`、`subprocess.DEVNULL`、现有文件描述符（正整数）、现有文件对象（open()函数返回的对象）和`None`。 
>
> 另外，*stderr*可以是 `subprocess.STDOUT`，表示应用程序的标准错误输出的文件句柄与标准输出的文件句柄相同。（相当于`2>&1`）

![](03python模块之subprocess\QQ截图20190411201111.png)

- **cwd**：如果*cwd*不是`None`，则在执行子进程程序之前先将工作目录更改为 *cwd*。
- 如果指定了*encoding*或*erroes*，或者*text*为`True`，则*stdin*，*stdout*和*stderr*的流为文本（str）流，*universal_newlines*参数等同于*text*，并提供向后兼容性。默认情况下，数据流为二进制流。

![](03python模块之subprocess\QQ截图20190411202330.png)

**Popen类的实例有如下方法：**

- `Popen.poll（）`：检查子进程是否已终止。
- `Popen.wait(timeout=None)`：挂起等待子进程结束，若设置了*timeout*参数，超时但进程还未结束则抛出`TimeoutExpired`异常。
- `Popen.communicate(input=None, timeout=None)`：与子进程进行交互，发送数据到标准输入、从标准输出或标准错误输出读取数据直到遇到EOF（end-of-file）。

> `communicate()`函数返回值为元组 (stdout_data, stderr_data)。
>
> 要使用该方法向子进程标准输入写入数据或读取标准输出和标准错误输出的数据，在实例化Popen类时，要指定`stdin=PIPE`、`stdout=PIPE`、`stderr=PIPE`。

- `Popen.send_signal(signal)`：发送信号到子进程。
- `Popen.terminate()`：终止子进程。

> On Posix OSs the method sends SIGTERM to the child. 
>
> On Windows the Win32 API function `TerminateProcess()` is called to stop the child.

- `Popen.kill()`

> Kills the child. On Posix OSs the function sends SIGKILL to the child. On Windows `kill()`is an alias for `terminate()`.

**Popen类的实例的属性：**

- `Popen.args`：获取*args*参数的值，是一个`list`或`str`。
- `Popen.stdin`：

> 如果实例化时，*stdin*参数是`PIPE`, 该属性会返回`open()`函数返回的可写的文件对象，该文件对象链接着子程序的标准输入。
>
> - 如果指定了*encoding*或*errors*参数或者*universal_newlines*参数为`True`, 文件流为文本（str）流, 否则为字节（byte）流。
>
> 如果实例化时，*stdin*参数不是`PIPE`, 该属性返回 `None`。

- `Popen.stdout`：

> 如果实例化时，*stdout*参数是`PIPE`, 该属性会返回`open()`函数返回的可读的文件对象，该文件对象链接着子程序的标准输出，读取该文件对象可以获得子程序的标准输出。
>
> - 如果指定了*encoding*或*errors*参数或者*universal_newlines*参数为`True`, 文件流为文本（str）流, 否则为字节（byte）流。
>
> 如果实例化时，*stdout*参数不是`PIPE`, 该属性返回 `None`。

- `Popen.stderr`：同上，但是读取的是子进程的标准错误输出。

![](03python模块之subprocess\QQ截图20190411195631.png)

- `Popen.pid`：获取子进程的PID。如果`shell=True`，返回则是系统shell进程的PID。
- `Popen.returncode`：返回码。

**使用`communicate()`而不是`.stdin.write`, `.stdout.read` or `.stderr.read`以避免由于管道缓冲区填满阻塞子进程而导致的死锁。** 

**上下文管理：**

> Popen对象支持`with`语句进行上下文管理。

```python
with Popen(["ifconfig"], stdout=PIPE) as proc:
    log.write(proc.stdout.read())
```

### 0x02 subprocess.run()

> 运行*args*描述的命令。等待命令执行完成，然后返回一个`CompletedProcess`类的实例。
>
> 参数的使用与Popen类基本相同。

```python
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, 
               cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None)
```

如果*capture_output* 为`True`, `stdout` 和 `stderr` 将会被捕获，默认为`False`。 

如果*shell*为`True`，表示使用系统的shell来执行*arg*描述的命令；如果为*shell*为`False`，则打开*arg*描述的对象。

![](03python模块之subprocess\QQ截图20190411161046.png)

如果指定了*encoding*或*erroes*，或者*text*为`True`，则*stdin*，*stdout*和*stderr*的流为文本（str）流，*universal_newlines*参数等同于*text*，并提供向后兼容性。默认情况下，数据流为二进制流。

![](03python模块之subprocess\QQ截图20190411161550.png)

### 0x04 subprocess.CompletedProcess()

> `subprocess.run()`方法执行结束后返回该类的实例。

**该类有如下方法（属性）：**

- **args**：获取传入`run()`方法的第一个参数，是一个`list`或`str`。
- **returncode**：获取返回码，为0表示函数执行成功。
- **stdout**：从标准输出中获取`run()`方法的输出结果（`capture_output=True`时才能获取到结果），结果为`bytes`类型字符串（若`encoding`参数不为`None`，则结果为`str`类型）。
- **stderr**：同上，但获取的是标准错误输出结果。
- **check_returncode（）**：当`run()`的返回值不为0时，调用该方法会抛出一个`CalledProcessError`异常。

**run()方法执行成功的情况：**

![](03python模块之subprocess\QQ截图20190411162626.png)

**run()方法执行失败的情况：**

![](03python模块之subprocess\QQ截图20190411164318.png)

**参考链接：**

https://docs.python.org/3/library/subprocess.html