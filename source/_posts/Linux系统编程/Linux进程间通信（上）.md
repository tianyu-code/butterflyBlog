---
title: Linux进程间通信（上）
date: 2020-08-04 14:36:32
tags:
  - Linux
  - 进程间通信
  - 管道
categories:
  - Linux系统编程

keywords: Linux,系统编程,进程间通信,管道,命名管道
cover: /images/封面图/p4.webp
top_img: /images/封面图/p4.webp
description: 本文介绍Linux进程间通信的概念以及管道的使用。
sticky: 0

---


#  简介

**进程间通信（IPC，InterProcess Communication）**是指在不同进程之间传播或交换信息。

IPC的方式通常有管道（包括无名管道和命名管道）、信号、消息队列、信号量、共享内存、套接字。
其中信号的介绍可查看该文[《Linux信号》](http://tianyu-code.top/Linux%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A8%8B/Linux%E4%BF%A1%E5%8F%B7/)

本文重点介绍管道的概念以及使用方式。

# 无名管道

## 概念

最基本的进程间通信方式，从本质上说，管道是一种特殊的文件，存在于内存中，但是它没有名字，只能用于有亲缘关系的进程间（fork）。

## 实现机制

管道是由内核管理的一个缓冲区，它的一端连接一个进程的输出，另一端连接一个进程的输入。管道的缓冲区不需要很大，它被设计为环形的数据结构，当两个进程都终止后，管道的生命周期也会被结束。

## 特点

1. 半双工，数据只能在一个方向上流动，若要双向通信，需要建立两个管道
2. 只能在具有公共祖先的进程之间使用，通常是`fork`的父子进程之间使用管道通信
3. 管道中的数据遵循先入先出的原则
4. 管道所传送的数据是无格式的，这要求管道的读出方与写入方必须事先约定好数据的格式
5. 管道不是普通的文件，不属于某个文件系统，其只存在于内存中，对应一块缓冲区，不同系统大小不一定相同
6. 管道数据被读走后抛弃，为了释放空间以便写更多数据


## 读写规则

1. 阻塞性：
    + 默认用`read`函数读数据是阻塞（无数据时）的，`write`函数也是阻塞的（当管道满了时）
    + 可通过`functl`来改变阻塞特性，如
        `funtl(fd,F_SETFL,0);`设为阻塞
        `funtl(fd,F_SETFL,0_NOBLOCK);`设为非阻塞
 
2. 退出：
    + 当写一个读端口已被关闭的管道时，写进程会收到SIGPIPE信号（默认退出，若从信号处理函数返回后则`write`函数出错返回，errno=EPIPE）
    + 当读一个写端口已被关闭的管道时，在所有数据都被读取后， `read`返回0，以指示达到了文件结束处


## 函数

```C
int pipe(int pipefd[2]);
功能：创建无名管道
参数：pipefd返回两个管道的文件描述符，fd[0]用于读，fd[1]用于写
返回值;成功返回0，失败返回-1，原因保存在errno中

```

常用方式如下：

<img src=/images/Linux进程间通信（上）/无名管道示意图.png>

代码如下：

```C
int
main(int argc, char *argv[])
{
   int pipefd[2];
   pid_t cpid;
   char buf;

   if (argc != 2) {
	   fprintf(stderr, "Usage: %s <string>\n", argv[0]);
	   exit(EXIT_FAILURE);
   }

   if (pipe(pipefd) == -1) {
	   perror("pipe");
	   exit(EXIT_FAILURE);
   }

   cpid = fork();
   if (cpid == -1) {
	   perror("fork");
	   exit(EXIT_FAILURE);
   }

   if (cpid == 0) {    /* Child reads from pipe */
	   close(pipefd[1]);          /* Close unused write end */

	   while (read(pipefd[0], &buf, 1) > 0)
		   write(STDOUT_FILENO, &buf, 1);

	   write(STDOUT_FILENO, "\n", 1);
	   close(pipefd[0]);
	   _exit(EXIT_SUCCESS);

   } else {            /* Parent writes argv[1] to pipe */
	   close(pipefd[0]);          /* Close unused read end */
	   write(pipefd[1], argv[1], strlen(argv[1]));
	   close(pipefd[1]);          /* Reader will see EOF */
	   wait(NULL);                /* Wait for child */
	   exit(EXIT_SUCCESS);
   }
}


```

# 命名管道（FIFO）

## 与无名管道的区别

1. 不相关的进程之间也能交换数据

2. 当使用FIFO的进程退出后，FIFO文件将继续保存在文件系统中以便以后使用

> 注意：创建命名管道一定要用指定函数创建，不能新建一个文件，因为命名管道是特殊的文件，如果对管道文件进行复制，会变成普通文件，不具有管道的特点

## 读写规则

1. 阻塞性：
    + 一般情况中（open时未指定O_NOBLOCK），当你用只读/只写方式打开FIFO的时候会阻塞，直到其他进程用只写/只读方式打开该FIFO
    + 如果指定了O_NOBLOCK，则只读打开立即返回。但是，如果没有进程已经为读而打开一个FIFO，那么只写打开将出错返回，其errno是ENXIO。


2. 退出：
    + 类似于管道，若写一个尚无进程为读而打开的FIFO，则产生信号SIGPIPE。
    + 若某个FIFO的最后一个写进程关闭了该FIFO，则将为该FIFO的读进程产生一个文件结束标志

## 函数

```C
int mkfifo(const char *pathname, mode_t mode);
功能：该函数创建一个命名管道的特殊文件pathname，mode指定了FIFO的读写权限（例如0777）
返回值：成功返回0，失败返回-1，失败原因保存在errno

```



---