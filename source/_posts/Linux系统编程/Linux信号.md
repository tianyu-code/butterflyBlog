---
title: Linux信号
date: 2020-08-03 12:36:32
tags:
  - Linux
  - 信号
categories:
  - Linux系统编程

keywords: Linux,系统编程,信号
cover: /images/封面图/Linux2.jpg
top_img: /images/封面图/Linux2.jpg
description: 本文介绍Linux中信号的概念和相关函数，以及线程与信号的使用。
sticky: 0

---


#  信号及信号来源

## 信号本质

信号是在软件层次上对中断机制的一种模拟，在原理上，一个进程收到一个信号与处理器收到一个中断请求可以说是一样的。信号是异步的，一个进程不必通过任何操作来等待信号的到达，事实上，进程也不知道信号到底什么时候到达。

信号是进程间通信机制中唯一的异步通信机制，可以看作是异步通知，通知接收信号的进程有哪些事情发生了。信号机制经过POSIX实时扩展后，功能更加强大，除了基本通知功能外，还可以传递附加信息。

## 信号来源

信号事件的发生有两个来源：硬件来源(比如我们按下了键盘或者其它硬件故障)；软件来源，最常用发送信号的系统函数是kill, raise, alarm和setitimer以及sigqueue函数，软件来源还包括一些非法运算等操作。

#  信号的种类


<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 100px;" bgcolor=#00a8f3>类型</th>
          <th align="center" valign="middle" style="width: 200px;" bgcolor=#00a8f3>内容</th>
          <th align="center" valign="middle" bgcolor=#00a8f3>说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>不可靠信号</td>
          <td>信号值小于SIGRTMIN的，这些信号是从早期的Unix继承来的</td>
          <td>问题：<br>
                1.每次处理完信号后，就对信号的响应设为默认值，但是该问题Linux对其改进，每次调用完信号处理函数后，不必重新安装<br>
                2.信号可能会丢失（不支持排队）</td>
      </tr>
      <tr>
          <td>可靠信号</td>
          <td>信号值在SIGRTMIN和SIGRTMAX之间的信号</td>
          <td>1.可靠信号的发送和注册函数变为sigqueue和sigaction<br>2.可排队</td>
      </tr>
      <tr>
          <td>实时信号和非实时信号</td>
          <td> </td>
          <td>非实时信号都不支持排队，都是不可靠信号<br>实时信号都支持排队，都是可靠信号</td>
      </tr>
    </tbody>
</table>

> 可使用`kill -l`查看全部信号

<img src=/images/Linux信号/Linux全部信号.png>

# 进程对信号的响应

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 100px;" bgcolor=#00a8f3>名称</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>含义</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>忽略信号</td>
          <td>即对信号不做任何处理，其中SIGKILL和SIGSTOP不可忽略</td>
      </tr>
      <tr>
          <td>捕捉信号</td>
          <td>信定义信号处理函数，当信号发生时执行相应的处理函数</td>
      </tr>
      <tr>
          <td>缺省操作</td>
          <td>执行信号默认的缺省操作，其中实时信号的缺省操作是进程终止</td>
      </tr>
    </tbody>
</table>


# 信号集及相关函数


## 相关概念介绍
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 100px;" bgcolor=#00a8f3>名称</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>含义</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>未决状态</td>
          <td>在信号产生(generation)和递送(delivery)之间(可能相当长)的时间间隔内，该信号处于未决(pending)状态，这种信号称为挂起(suspending)的信号</td>
      </tr>
      <tr>
          <td>未决（未处理的）信号队列</td>
          <td>内核为每个进程维护一个未决（未处理的）信号队列，信号产生时无论是否被阻塞，首先放入未决队列里。当时间片调度到当前进程时，内核检查未决队列中是否存在信号。若有信号且未被阻塞，则执行相应的操作并从队列中删除该信号；否则仍保留该信号</td>
      </tr>
      <tr>
          <td rowspan="2">信号屏蔽字（信号阻塞集）</td>
          <td>每个进程都有一个信号屏蔽字(signal mask)，规定当前要阻塞递送到该进程的信号集。对于每个可能的信号，该屏蔽字中都有一位与之对应。对于某种信号，若其对应位已设置，则该信号当前被阻塞</td>
      </tr>
      <tr>
          <td bgcolor=#03bb03> 所谓阻塞并不是禁止传送信号, 而是暂缓信号的传送。若将被阻塞的信号从信号阻塞集中删除，且对应的信号在被阻塞时发生了，进程将会收到相应的信号</td>
      </tr>
    </tbody>
</table>

## 信号集

信号集用来描述信号的集合，linux所支持的所有信号可以全部或部分的出现在信号集中，主要与信号阻塞相关函数配合使用

```C
结构定义：
typedef struct {
    unsigned long sig[_NSIG_WORDS]；
} sigset_t

//初始化由set指定的信号集，信号集里面的所有信号被清空
int sigemptyset(sigset_t *set);

//调用该函数后，set指向的信号集中将包含linux支持的64种信号
int sigfillset(sigset_t *set);

//判定信号signum是否在set指向的信号集中
int sigismember(const sigset_t *set, int signum);

//在set指向的信号集中加入signum信号
int sigaddset(sigset_t *set, int signum);

//在set指向的信号集中加入signum信号
int sigdelset(sigset_t *set, int signum);

//以上函数成功返回0，失败返回-1

```

## 信号阻塞

每个进程都有一个用来描述哪些信号递送到进程时将被阻塞的信号集，该信号集中的所有信号在递送到进程后都将被阻塞

```C
//根据how参数来对信号集进行对应的操作:
//SIG_BLOCK：在进程当前屏蔽信号集中添加set指向信号集中的信号
//SIG_UNBLOCK：如果进程屏蔽信号集中包含set指向信号集中的信号，则解除对该信号的阻塞
//SIG_SETMASK：更新进程屏蔽信号集为set指向的信号集

//若oldset不为NULL，则将旧的信号阻塞集通过该参数返回
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

//获得当前进程的未决信号队列
int sigpending(sigset_t *set);

//挂起线程且暂时使用mask代替当前信号阻塞集，直到信号到达
//1.若信号的处理是终止进程，则该函数不返回
//2.若注册了信号处理函数，则待信号处理函数执行完毕后，该函数再返回
int sigsuspend(const sigset_t *mask);

//以上函数成功返回0，失败返回错误码
```

---
# 信号发送函数

## kill

```C
int kill(pid_t pid, int sig);
功能：给任意进程发送任意信号
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 400px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="2">参数</td>
          <td>pid</td>
          <td>(1)若pid > 0,则发送信号给pid的进程<br>
                (2)若pid = 0，则发送信后给本进程组的所有进程<br>
                (3)若pid = -1，则发送信号到调用进程有权发送信号的每个进程，init进程除外<br>
                (4)若pid < -1,则发送信号给进程组ID为-pid中的每个进程</td>
          <td rowspan="3" >若sig=0，则不发送任何信号，但是参数检测仍然进行，这可以用来检查pid参数是否正确（进程是否存在或允许发送信号）</td>
      </tr>
      <tr>
          <td>sig</td>
          <td>待发送信号</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1，错误值在errno中</td>
      </tr>
    </tbody>
</table>


## sigqueue

```C
int sigqueue(pid_t pid, int sig, const union sigval value);
功能：给任意进程发送任意信号，并且可以传递数据
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
          <td>pid</td>
          <td>则发送信号给pid的进程</td>
          <td rowspan="4" >1.该函数是比较新的发送信号系统调用，主要针对实时信号提出的，也支持前32中，常常配合sigaction一起使用<br>
                            2.该函数和kill类似，但只能发送给一个进程，不能发送给进程组，当sig=0时的行为和kill一致<br>
                            3.该函数可在发送信号时传递数据给3参数信号处理函数，通过value参数进行，定义如下：<br>
                            union sigval {<br>
                            &nbsp&nbsp&nbsp&nbsp int   sival_int;<br>
                            &nbsp&nbsp&nbsp&nbsp void *sival_ptr;<br>
                            };</td>
      </tr>
      <tr>
          <td>sig</td>
          <td>待发送信号</td>
      </tr>
      <tr>
          <td>value</td>
          <td>随信号一起传递的数据</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1，错误值在errno中</td>
      </tr>
    </tbody>
</table>

## raise
```C
int raise(int sig);
功能：给本进程或线程发送任意信号
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
          <td align="center" valign="middle">参数</td>
          <td>sig</td>
          <td>待发送信号</td>
          <td rowspan="2" >1.在单线程程序中等价于kill(getpid(), sig);<br>
                            2.在多线程程序中等价于pthread_kill(pthread_self(), sig);<br>
                            3.该函数会在信号处理函数执行完成后返回</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回非0</td>
      </tr>
    </tbody>
</table>

## alarm
```C
unsigned int alarm(unsigned int seconds);
功能：在seconds秒后给本进程发送SIGALRM信号
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 430px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle">参数</td>
          <td>seconds</td>
          <td>时间</td>
          <td rowspan="2" >1.alarm默认处理是终止进程<br>
                            2.若seconds=0，则任何未决的alarm都会被取消<br>
                            3.注意：这个函数是无阻塞的</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>unsigned int</td>
          <td>如果以前没有设置过alarm或者已经超时，那么返回0<br>
                如果以前设置过alarm，那就返回剩余的时间，并且重新设定定时器</td>
      </tr>
    </tbody>
</table>

## abort

```C
void abort(void);
功能：给本进程发送SIGABRT信号
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 200px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle">参数</td>
          <td>void</td>
          <td>无</td>
          <td rowspan="2" >1.该函数先解除对SIGABRT信号的屏蔽<br>
2.不论该信号被屏蔽或是注册了信号处理函数，它总会终止进程，该函数通过回复SIGABRT的默认配置然后再次发出该信号来完成此操作。（除非你未从信号处理函数返回（see longjump）</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>void</td>
          <td>无</td>
      </tr>
    </tbody>
</table>


---
# 信号处理函数

## signal

```C
typdef void (*sighandler_t )( int );
sighandler_t signal ( int  signum, sighandler_t  handler);
功能：注册简单的信号处理函数
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 400px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="2">参数</td>
          <td>signum</td>
          <td>待注册的信号</td>
          <td rowspan="3" >注意：SIGKILL和SIGSTOP不可注册处理函数</td>
      </tr>
      <tr>
          <td>handler</td>
          <td>注册的信号处理函数，可填写：<br>SIG_IGN:忽略该信号<br>SIG_DFL:系统默认方式处理该信号</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>sighandler_t</td>
          <td>成功返回上一次安装的信号处理函数（若是第一次注册返回NULL），失败返回SIG_ERR，错误值存在errno中</td>
      </tr>
    </tbody>
</table>

## sigaction

```C
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
功能：注册信号处理函数，该函数可具备3个参数且可接受附加参数
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
          <td>signum</td>
          <td>待注册的信号</td>
          <td rowspan="4" >act指定了相关的处理函数,定义如下：<br>
                        struct sigaction {<br>
                        &nbsp&nbsp&nbsp&nbsp void     (*sa_handler)(int);<br>
                        &nbsp&nbsp&nbsp&nbsp void     (*sa_sigaction)(int, siginfo_t *, void *);<br>
                        &nbsp&nbsp&nbsp&nbsp sigset_t   sa_mask;<br>
                        &nbsp&nbsp&nbsp&nbsp int        sa_flags;<br>
                        &nbsp&nbsp&nbsp&nbsp void     (*sa_restorer)(void);<br>
                        };<br>
                        该结构体中sa_restorer已经过时不用了，<br>
                        （1）sa_handler还是以前的那种信号处理函数，参数只有信号值<br>
                        （2）sa_sigaction是3参数信号处理函数，下面介绍<br>
                        （3）sa_mask指定在信号处理函数中，屏蔽那些信号，缺省情况下是当前信号，防止信号的嵌套<br>
                        （4）sa_flags，该参数中指定了一些标志位：<br>
                            SA_NODEFER/SA_NOMASK：指定在信号处理函数中不将本信号加入阻塞集，后者是过时的同义参数<br>
                            SA_SIGINFO：表示信号附带的参数可以被传递到信号处理函数，使用3参数处理函数中必须将该位置位<br>
                        其余此处暂不介绍<br>
                        注意：信号处理函数只能注册其中一个<br></td>
      </tr>
      <tr>
          <td>act</td>
          <td>注册的信号处理函数</td>
      </tr>
      <tr>
          <td>oldact</td>
          <td>返回旧的处理函数，可填写NULL</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1，错误值在errno中</td>
      </tr>
    </tbody>
</table>

## 3参数信号处理函数sa_sigaction

```C
void (*sa_sigaction)(int, siginfo_t *, void *);

//参数：sig，info，ucontext
//sig是信号值，第三个参数未使用，info参数结构体定义如下：
typedef struct {
    int si_signo;
    int si_errno;           
    int si_code;            
    union sigval si_value;  
    } siginfo_t;
//这个结构体中的第四个参数，就是sigqueue给信号传递的那个结构体了
```

# 可重入函数

为了增强程序的稳定性，在信号处理函数中应使用`可重入函数`。因为进程在收到信号后，就将跳转到信号处理函数去接着执行。如果信号处理函数中使用了不可重入函数，那么信号处理函数可能会修改原来进程中不应该被修改的数据，这样进程从信号处理函数中返回接着执行时，可能会出现不可预料的后果。不可再入函数在信号处理函数中被视为不安全函数。

> 所谓可重入函数是指一个可以被多个任务调用的过程，任务在调用时不必担心数据是否会出错

如何编写可重入函数：
1. 不使用（返回）静态的数据、全局变量（除非用信号量互斥）。
2. 不调用动态内存分配、释放的函数。
3. 不调用任何不可重入的函数（如标准I/O函数）

> 即使信号处理函数使用的都是可重入函数（常见的可重入函数），也要注意进入处理函数时，首先要保存errno的值，结束时，再恢复原值。因为，信号处理过程中，errno值随时可能被改变。


---