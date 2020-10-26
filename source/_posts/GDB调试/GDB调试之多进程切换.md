---
title: GDB调试之多进程/线程
date: 2020-04-07 23:10:52
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, 多进程/线程
description: 本文介绍GDB调试中多进程和多线程的操作

---


#  选择调试进程

在GDB中有两个选项来确定调试的进程：
+ `follow-fork-mode`，设置调试哪个进程
+ `detach-on-fork`，GDB在fork之后是否断开（detach）某个进程的调试

这两个选项的参数组合起来的效果如下表

follow-fork-mode | detach-on-fork | 效果
--- | --- | --- 
parent | on | 只调试父进程
child | on | 只调试子进程
parent | off | 同时调试两个进程，子进程暂停
child | off | 同时调试两个进程，子进程暂停


#  进程切换

+ `info inferiors`，查看当前所有进程
+ `inferiors <num>`，切换当前GDB调试进程，其中`num`为上一条指令中列出的进程Num

# 实例

```
int main(void)
{
    pid_t pid;
    pid = fork();
    if(pid < 0)
    {
        printf("fork error\n");
    }
    else if(pid == 0)
    {
        printf("in child \n");
    }
    else
    {
        printf("in father,pid of child:%d\n", pid);

    }

}

```

首先展示如何选择跟踪父子进程，如图

<image src=/images/GDB调试之多进程切换/跟踪父子进程.png>


若同时调试两个进程，并且切换进程的效果如下

<image src=/images/GDB调试之多进程切换/切换进程.png>

# 多线程调试

在另一篇博客[《GDB调试之基本指令介绍》](http://tianyu-code.top/GDB%E8%B0%83%E8%AF%95/GDB%E8%B0%83%E8%AF%95%E4%B9%8B%E5%9F%BA%E6%9C%AC%E6%8C%87%E4%BB%A4%E4%BB%8B%E7%BB%8D/)中第6章提到了在线程中打断点，这里再介绍下
> 当你的程序是多线程时，你可以定义你的断点是否在所有的线程上，或是在某个特定的线程。
`break line thread threadNo`
其中`line`为你的源码行数，threadNo为`info threads`命令中GDB给出的线程ID，若不指定`threadNo`，则为所有线程打断点。

1. 在多线程调试时，可以设置其余线程的**阻塞状态**
    + `show scheduler-locking`，查看设置
    + `set scheduler-locking <on><off><step>`
        + `on`，表示调试线程执行时，其余线程锁定，阻塞等待，
        + `off` ，表示不锁定其他线程
        + `step` ，表示在step（单步）调试时，只有当前线程运行

    这样就可以避免next调试时总是跳转到其他线程啦

2. 线程和进程一样，同样支持**切换**
    + `info thread`, 列出当前所有线程
    + `thread <num>`，切换线程，num为上一条指令给出的



