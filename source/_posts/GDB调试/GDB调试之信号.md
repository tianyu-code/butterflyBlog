---
title: GDB调试之信号
date: 2020-03-29 23:11:06
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, 信号
description: 本文介绍在GDB调试中对于信号的操作

---

# 目录

+ GDB发送信号
+ GDB对信号的处理
+ 实例

<!--------more------->

# **GDB发送信号**

在GDB调试状态中，可以在命令号输入`signal 信号`来向程序发送信号

# **GDB处理信号**

> 背景：GDB调试时，能够捕获产生的信号并停止，当频繁有信号产生时，很影响调试

在GDB中handle指令用于设置GDB对于信号的处理，可以输入`help handle`来查看

```
Specify how to handle signals.
Usage: handle SIGNAL [ACTIONS]
Args are signals and actions to apply to those signals.
If no actions are specified, the current settings for the specified signals
will be displayed instead.
Symbolic signals (e.g. SIGSEGV) are recommended but numeric signals
from 1-15 are allowed for compatibility with old versions of GDB.
Numeric ranges may be specified with the form LOW-HIGH (e.g. 1-5).
The special arg "all" is recognized to mean all signals except those
used by the debugger, typically SIGTRAP and SIGINT.
Recognized actions include "stop", "nostop", "print", "noprint",
"pass", "nopass", "ignore", or "noignore".
Stop means reenter debugger if this signal happens (implies print).
Print means print a message if this signal happens.
Pass means let program see this signal; otherwise program doesn't know.
Ignore is a synonym for nopass and noignore is a synonym for pass.
Pass and Stop may be combined.
Multiple signals may be specified.  Signal numbers and signal names
may be interspersed with actions, with the actions being performed for
all signals cumulatively specified.

```
总结大体意思：
+ **nostop**
当被调试的程序收到信号时，GDB不会停住程序的运行，但会打出消息告诉你收到这种信号。
+ **stop**
当被调试的程序收到信号时，GDB会停住你的程序。
+ **print**
当被调试的程序收到信号时，GDB会显示出一条信息。
+ **noprint**
当被调试的程序收到信号时，GDB不会告诉你收到信号的信息。
+ **pass** 、**noignore**
当被调试的程序收到信号时，GDB不处理信号。这表示，GDB会把这个信号交给被调试程序会处理。
+ **nopass**、**ignore**
当被调试的程序收到信号时，GDB不会让被调试程序来处理这个信号

在GDB中可以使用`info signals`和`info handle`来查看有哪些信号在被GDB检测中和GDB对其的处理

```
Signal        Stop      Print   Pass to program Description

SIGHUP        Yes       Yes     Yes             Hangup
SIGINT        Yes       Yes     No              Interrupt
SIGQUIT       Yes       Yes     Yes             Quit
SIGILL        Yes       Yes     Yes             Illegal instruction
SIGTRAP       Yes       Yes     No              Trace/breakpoint trap
SIGABRT       Yes       Yes     Yes             Aborted
SIGEMT        Yes       Yes     Yes             Emulation trap
SIGFPE        Yes       Yes     Yes             Arithmetic exception
SIGKILL       Yes       Yes     Yes             Killed
SIGBUS        Yes       Yes     Yes             Bus error
SIGSEGV       Yes       Yes     Yes             Segmentation fault
...
SIGUSR1       Yes       Yes     Yes             User defined signal 1
SIGUSR2       Yes       Yes     Yes             User defined signal 2

还有很多不列出。
```


# **实例**

例程代码如下
```
void signalHandle(int para)
{
    printf("in signal handle,signal num:%d\n", para);
}

void signalHandle2(int para)
{
    printf("in signal handle,signal num:%d\n", para);
    exit(0);
}


int main(void)
{
    signal(SIGUSR1, signalHandle);
    signal(SIGUSR2, signalHandle2);

    while(1)
    {
        printf("press any key to send a signal\n");
        getchar();
        raise(SIGUSR1);
     }

    return 0;
}
```
为了效果更佳明显，先运行程序，然后gdb attach到进程上，进入gdb调试模式
```
(gdb) c
Continuing.

Program received signal SIGUSR1, User defined signal 1.
0x00007fd1f1ec9337 in raise () from /lib64/libc.so.6
(gdb) c
Continuing.

```
可以发现当在程序中按下回车时，进程给自己发送了SIGUSR1信号，并且GDB捕获到该信号切停下，我们继续运行，
<font color=red>将GDB对SIGUSR1信号的处理设置为nostop</font>


```
Program received signal SIGUSR1, User defined signal 1.
0x00007fd1f1ec9337 in raise () from /lib64/libc.so.6
(gdb) handle SIGUSR1 nostop
Signal        Stop	Print	Pass to program	Description
SIGUSR1       No	Yes	Yes		User defined signal 1
(gdb) c
Continuing.

Program received signal SIGUSR1, User defined signal 1


```
可以发现此时只会打印，而gdb不会停下，接下来
<font color=red>将GDB对SIGUSR1信号的处理设置为noprint</font>

```
(gdb) handle SIGUSR1 noprint
Signal        Stop	Print	Pass to program	Description
SIGUSR1       No	No	Yes		User defined signal 1
(gdb) c
Continuing.
```
此时GDB既不会打印也不会停下

通过GDB给程序发送信号
```
(gdb) singal SIGUSR2
Continuing with signal SIGUSR2.
[Inferior 1 (process 20693) exited normally]
```
程序命令行打印in signal handle,signal num:12,显然收到SIGUSR2信号
