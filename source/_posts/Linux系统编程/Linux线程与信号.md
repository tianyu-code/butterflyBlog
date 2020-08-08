---
title: Linux线程与信号
date: 2020-08-04 12:36:32
tags:
  - Linux
  - 信号
  - 线程与信号
categories:
  - Linux系统编程

keywords: Linux,系统编程,信号,线程与信号
cover: /images/封面图/Linux.jfif
top_img: /images/封面图/Linux.jfif
description: 本文介绍Linux中信号的概念和相关函数，以及线程与信号的使用。
sticky: 0

---


# 线程与信号

> 在Linux的多线程中使用信号机制，与在进程中使用信号机制有着根本的区别，可以说是完全不同。

在`进程环境`中，对信号的处理是:先注册信号处理函数，当信号异步发生时，调用处理函数来处理信号。它`完全是异步的`（我们完全不知到信号会在进程的那个执行点到来！）。然而信号处理函数的实现，`有着许多的限制`；比如有一些函数不能在信号处理函数中调用；再比如一些函数read、recv等调用时会被异步的信号给中断(interrupt)，因此我们必须对在这些函数在调用时因为信号而中断的情况进行处理（判断函数返回时 enno 是否等于 EINTR）

但是在`多线程`中处理信号的原则却完全不同，它的基本原则是：`将对信号的异步处理，转换成同步处理`。也就是说用一个线程专门的来`“同步等待”`信号的到来，而其它的线程可以完全不被该信号中断/打断(interrupt)。这样就在相当程度上简化了在多线程环境中对信号的处理。而且可以保证其它的线程不受信号的影响。这样我们对信号就可以完全预测，因为它不再是异步的，而是同步的（我们完全知道信号会在哪个线程中的哪个执行点到来而被处理！）。而同步的编程模式总是比异步的编程模式简单。其实多线程相比于多进程的其中一个优点就是：多线程可以将进程中异步的东西转换成同步的来处理

> 记住：在多线程代码中，总是使用sigwait或者sigwaitinfo或者sigtimedwait等函数来处理信号，而不是signal或者sigaction等函数

这种采用专用线程同步处理信号的模型如下图所示: 

<img src=/images/Linux线程与信号/线程与信号.png>

其设计步骤如下：

1. 主线程设置信号屏蔽字，阻塞希望同步处理的信号；

2. 主线程创建一个信号处理线程，该线程将希望同步处理的信号集作为 sigwait()的参数；

3. 主线程创建若干工作线程。

主线程的信号屏蔽字会被其创建的新线程继承，故工作线程将不会收到信号。

注意:因程序逻辑需要而产生的信号(如SIGUSR1/ SIGUSR2和实时信号)，被处理后程序继续正常运行，可考虑使用sigwait同步模型规避信号处理函数执行上下文不确定性带来的潜在风险。而对于硬件致命错误等导致程序运行终止的信号(如SIGSEGV)，必须按照传统的异步方式使用 signal()或sigaction()注册信号处理函数进行非阻塞处理，以提高响应的实时性。在应用程序中，可根据所处理信号的不同而同时使用这两种信号处理模型。

---
# 与进程的区别


<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 100px;" bgcolor=#00a8f3>名称</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>含义</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>线程的未决队列</td>
          <td>内核也为每个线程维护未决信号队列。当调用sigpending()时，返回整个进程未决信号队列与调用线程未决信号队列的并集</td>
      </tr>
      <tr>
          <td>线程的未决队列和屏蔽字的继承</td>
          <td>1.进程内创建线程时，新线程将继承进程(主线程)的信号屏蔽集，但新线程的未决信号集被清空(以防同一信号被多个线程处理)。<br>
                2.线程的信号屏蔽字是私有的(定义当前线程要求阻塞的信号集)，即线程可独立地屏蔽某些信号。这样，应用程序可控制哪些线程响应哪些信号</td>
      </tr>
      <tr>
          <td>信号处理函数</td>
          <td>1.信号处理函数由进程内所有线程共享。即对某个信号处理函数，以最后一次注册的处理函数为准，从而保证同一信号被任意线程处理时行为相同。<br>
                2.此外，若某信号的默认动作是停止或终止，则不管该信号发往哪个线程，整个进程都会停止或终止<br>
                3.当信号到来时，由哪个线程去处理是不确定的</td>
      </tr>
    </tbody>
</table>

---
# 函数介绍

## sigwait

```C
int sigwait(const sigset_t *set, int *sig);
功能：阻塞等待信号
```
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 250px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="2">参数</td>
          <td>set</td>
          <td>等待的信号集</td>
          <td rowspan="3" >1.该函数阻塞直到set集合中的任意信号变为未决状态（若已存在未决信号，则该函数立即返回），然后接收信号（将其从未决队列中删除），并通过*sig返回信号值<br>
                            2.若同时有多个等待中的信号处于未决状态，则对这些信号的选择规则和顺序未定义<br>
                            3.注意：他是接收未决状态的信号，那么信号就算被屏蔽了，也可以接收到<br>
                            4.注意：该函数不会被信号打断<br>
                            5.多个线程调用sigwait()等待同一信号，只有一个(但不确定哪个)线程可从sigwait()中返回。若信号被捕获(通过sigaction安装信号处理函数)，且线程正在sigwait()调用中等待同一信号，则由系统实现来决定以何种方式递送信号。操作系统实现可让sigwait返回(通常优先级较高)，也可激活信号处理程序，但不可能出现两者皆可的情况
                            </td>
      </tr>
      <tr>
          <td>sig</td>
          <td>返回等待到的信号</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回信号值，失败返回-1，错误信息保存在errno中</td>
      </tr>
    </tbody>
</table>

> 默认忽略对SIGKILL和SIGSTOP的等待

## sigwaitinfo
```C
int sigwaitinfo(const sigset_t *set, siginfo_t *info);
功能：阻塞等待信号，并返回相关数据,其实该函数就是sigwait的升级版，可接收sigqueue发送信号附带的数据
```
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 300px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="2">参数</td>
          <td>set</td>
          <td>等待的信号集</td>
          <td rowspan="3" >1.该函数与sigwait的功能相似，区别在于返回值不同且能接收信号附带的数据<br>
                            2. 该函数能被信号打断</td>
      </tr>
      <tr>
          <td>info</td>
          <td>若该参数不为NULL，则返回相关数据（该结构体就是3参数信号处理函数的第二个参数）</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回信号值，失败返回-1，错误信息保存在errno中</td>
      </tr>
    </tbody>
</table>


## sigtimedwait
```C
int sigtimedwait(const sigset_t *set, siginfo_t *info, const struct timespec *timeout);
功能：阻塞等待信号，并返回相关数据，并提供超时退出
```
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 300px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>set</td>
          <td>等待的信号集</td>
          <td rowspan="4" >该函数其实和sigwaitinfo一样，只是提供了超时退出参数，timeout指定了线程被挂起等待信号的时间间隔，定义如下：<br>
                            struct timespec {<br>
                            &nbsp&nbsp&nbsp&nbsplong    tv_sec;         /* seconds */<br>
                            &nbsp&nbsp&nbsp&nbsplong    tv_nsec;        /* nanoseconds */<br>
                            }<br>
                            若timeout结构体中两个参数均写0，则执行轮询：timedwait（）立即返回，其中包含有关调用方未决信号的信息，或者如果set中没有信号未决，则返回错误。</td>
      </tr>
      <tr>
          <td>info</td>
          <td>若该参数不为NULL，则返回相关数据（该结构体就是3参数信号处理函数的第二个参数）</td>
      </tr>
      <tr>
          <td>timeout</td>
          <td>超时退出参数</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回信号值，失败返回-1，错误信息保存在errno中</td>
      </tr>
    </tbody>
</table>

## pthread_sigmask
```C
int pthread_sigmask(int how, const sigset_t *set, sigset_t *oldset);
功能：设置本线程的信号屏蔽集
```
> 该函数和[sigprocmask](http://tianyu-code.top/Linux%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A8%8B/Linux%E4%BF%A1%E5%8F%B7/)一样，区别就是该函数用于多线程，设置当前线程自己的屏蔽集
## pthread_kill
```C
int pthread_kill(pthread_t thread, int sig);
功能：给本进程的其他线程发送信号，该信号是异步送达的
thread:线程ID
sig:待发送信号
成功返回信号值，失败返回错误码
```

# 工作小结

突然想到之前写的程序中，并未采用本文中的方式在多线程程序中使用信号，结果发现了信号处理函数重入的现象，当时不理解，学习了线程与信号，反思如下：

1. 每个线程的信号屏蔽集是独立的，那么信号处理函数中，会将本次的信号加入屏蔽集，是加入了谁的屏蔽集呢？
2. 若对进程发送信号，那么该进程中存在多个线程，则当前哪个线程在运行，则哪个线程被中断进行信号处理函数，那是不是将本信号加入了这个线程的屏蔽集
3. 若还存在其他线程，则此时其他线程调度运行了，那么再次产生该信号，会不会就发生这种信号处理函数的重入现象。

进行试验发现，使用另外一个进程快速的发送信号，线程处理函数重入的次数，和线程数量有关，基本验证了猜想


---