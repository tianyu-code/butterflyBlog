---
title: Linux中fcntl函数介绍
date: 2020-08-08 14:36:32
tags:
  - Linux
  - fcntl
categories:
  - Linux系统编程

keywords: Linux,系统编程,文件IO,文件描述符,fcntl
cover: /images/封面图/shell.jfif
top_img: /images/封面图/shell.jfif
description: 本文介绍文件描述符的控制函数fcntl的使用方法。
sticky: 0

---


# 函数简介

```C
int fcntl(int fd, int cmd, ... /* arg */ )
功能描述：对文件描述符提供控制，针对cmd的值,fcntl能够接受第三个参数
返回值根据不同参数不同，失败均返回-1
```

fcntl函数有5种功能：

1. 复制一个现有的描述符（cmd=F_DUPFD）.
2. 获得／设置文件描述符标记(cmd=F_GETFD或F_SETFD).
3. 获得／设置文件状态标记(cmd=F_GETFL或F_SETFL).
4. 获得／设置异步I/O所有权(cmd=F_GETOWN或F_SETOWN).
5. 获得／设置记录锁(cmd=F_GETLK,F_SETLK或F_SETLKW).


# 功能/参数详解

## 复制一个现有的描述符

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 200px;" >描述</th>
          <th align="center" valign="middle" >说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>cmd = F_DUPFD</td>
          <td>复制文件描述符，使用大于或等于arg的编号最小的可用文件描述符复制文件描述符fd。 这与dup2不同，dup2完全使用指定的文件描述符</td>
      </tr>
      <tr>
          <td>cmd = F_DUPFD_CLOEXEC</td>
          <td>操作和F_DUPFD一样，但是会将新fd的close-on-exec置位</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>新fd的最小值</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>获得的新fd</td>
      </tr>
    </tbody>
</table>



## 获得／设置文件描述符标记


<table>
    <thead>
      <tr>
          <th align="center" valign="middle" >描述</th>
          <th align="center" valign="middle" >说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>cmd = F_GETFD</td>
          <td>取得与文件描述符关联的标志,目前仅定义了close-on-exec</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>无</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>与文件描述符关联的标志</td>
      </tr>
      <tr>
          <td></td>
          <td></td>
      </tr>
      <tr>
          <td>cmd = F_SETFD</td>
          <td>设置与文件描述符关联的标志:<br>
                flags |= FD_CLOEXEC; //打开标志位<br>
                flags &= ~FD_CLOEXEC; //关闭标志位</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>新flag</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>0</td>
      </tr>
    </tbody>
</table>





## 获得／设置文件状态标记


<table>
    <thead>
      <tr>
          <th align="center" valign="middle" >描述</th>
          <th align="center" valign="middle" >说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>cmd = F_GETFL</td>
          <td>取得fd的文件状态标志，状态标志请参考open的flag参数</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>无</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>与文件描述符关联的文件状态标志</td>
      </tr>
      <tr>
          <td></td>
          <td></td>
      </tr>
      <tr>
          <td>cmd = F_SETFL</td>
          <td>设置fd的文件状态标志，该命令仅能更改O_APPEND，O_ASYNC，O_DIRECT，O_NOATIME和O_NONBLOCK标志</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>新flag</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>0</td>
      </tr>
    </tbody>
</table>

## 获得／设置异步I/O所有权

> 即信号管理功能，文件可能为设备，在设备驱动中尝试用信号SIGIO来实现和应用程序的异步通信

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" >描述</th>
          <th align="center" valign="middle" >说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>cmd = F_GETOWN</td>
          <td>获取当前在文件描述符fd上接收SIGIO和SIGURG信号的进程ID或进程组ID</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>无</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>进程ID以正值形式返回； 进程组ID作为负值返回</td>
      </tr>
      <tr>
          <td></td>
          <td></td>
      </tr>
      <tr>
          <td>cmd = F_SETOWN</td>
          <td>设置将接收SIGIO和SIGURG信号的进程id或进程组id,进程组id通过提供负值的arg来说明<br>
          注意：除了设置接收SIGIO的进程ID外，还需要设置文件描述符的O_ASYNC标志位才能正确使用信号</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>进程ID或进程组ID（负值）</td>
      </tr>
      <tr>
          <td>返回值</td>
          <td>0</td>
      </tr>
    </tbody>
</table>


> 补充：信号管理功能还有两个参数F_GETSIG/F_SETSIG，但是网上资料较少，只能贴上Linux的man手册原文以及自己的理解

个人理解：

+ F_GETSIG：获取异步通信的信号值，即当输入或输出变为可能时，用哪个信号进行异步通信,默认为SIGIO
+ F_SETSIG：设置异步通信的信号值，即当输入或输出变为可能时，设置指定的信号用于异步通信


原文如下(谷歌机翻)：

```
F_GETSIG

Return  (as the function result) the signal sent when input or output becomes possible.  
A value of zero means SIGIO is sent.  Any other value (including SIGIO) is the signal sent instead,  
and  in  this  case additional info is available to the signal handler if installed with SA_SIGINFO

当输入或输出变为可能时，返回（作为功能结果）发送的信号。零值表示已发送SIGIO。
其他任何值（包括SIGIO）是发送的信号，在这种情况下，如果与SA_SIGINFO一起安装，则其他信息可用于信号处理程序


F_SETSIG

Set  the  signal  sent when input or output becomes possible to the value given in arg. 
A value of zero means to send the default SIGIO signal.  Any other  value  (including  SIGIO)  
is  the  signal  to  send instead,  and  in this case additional info is available to the 
signal handler if installed with SA_SIGINFO

将输入或输出变为可能时发送的信号设置为arg中给出的值。零值表示发送默认SIGIO信号。 
其他任何值（包括SIGIO）将代替发送信号，在这种情况下，如果与SA_SIGINFO一起安装，则其他信息可用于信号处理程序

By using F_SETSIG with a nonzero value, and setting  SA_SIGINFO  for  the  signal  handler  (see  sigaction(2)),  
extra information about I/O events is passed to the handler in a siginfo_t structure.  If the si_code field 
indicates the source is SI_SIGIO, the si_fd field gives  the  file  descriptor  associated with  the  event.  
Otherwise, there is no indication which file descriptors are pending, and you should use the usual 
mechanisms (select(2), poll(2), read(2) with O_NONBLOCK set etc.) to determine which  
file descriptors are available for I/O

通过使用具有非零值的F_SETSIG并为信号处理程序设置SA_SIGINFO（请参阅sigaction（2）），
有关I / O事件的其他信息将以siginfo_t结构传递给处理程序。 如果si_code字段指示源是SI_SIGIO，
则si_fd字段提供与事件关联的文件描述符。 否则，没有指示哪些文件描述符正在挂起，您应该使用通常的
机制（select（2），poll（2），read（2）以及O_NONBLOCK设置等）来确定哪些文件描述符可用于I /O



Note that the file descriptor provided in si_fd is the one that was specified during the F_SETSIG operation.  
This can lead to an unusual corner case.  If the file descriptor is duplicated (dup(2)  or  similar), 
and the original file descriptor is closed, then I/O events will continue to be generated, but the 
si_fd field will contain the number of the now closed file descriptor

请注意，si_fd中提供的文件描述符是在F_SETSIG操作期间指定的文件描述符。 这可能会导致异常情况。 
如果文件描述符重复（dup（2）或类似文件），并且关闭了原始文件描述符，则将继续生成I / O事件，
但是si_fd字段将包含现在关闭的文件描述符的编号

```


## 获得／设置记录锁

### 概念

当两个人同时编辑一个文件时，其后果将如何呢？在很多UNIX系统中，该文件的最后状态取决于写该文件的最后一个进程。但是对于有些应用程序，例如数据库，有时进程需要确保它正在单独写一个文件。为了向进程提供这种功能，较新的UNIX系统提供了记录锁机制。

**记录锁（record locking）**的功能是：一个进程正在读或修改文件的某个部分时，可以阻止其他进程修改同一文件区

### 参数

操作记录锁时，函数原型如下

```C
int fcnt1(int fd,int cmd,.../* struct flock * flockptr */ )
```

1. `struct flock`结构体

    在操作记录锁时，传入的arg参数为`struct flock`结构体，定义如下

    ```C
    struct flock{
        short l_type;  // F_RDLCK读锁，F_WRLCK写锁，F_UNLCK解锁
        off_t l_start; // 锁偏移的字节
        short l_whence;// 锁开始的位置，SEEK_SET,SEEK_CURorSEEK_END
        off_t l_len;   // 锁的范围，0表示从起始到文件末
        pid_t l_pid;   // 占有锁的进程ID(仅由F_GETLK返回)
    };

    ```
    **注意：**
    + len单位时字节，0时表示从起始位置到文件末尾，就表示之后在解锁前加入的内容都在锁区域。
    + 通常对通篇文件上锁的参数是 l_start = 0; l_whence = SEEK_SET ; l_len = 0;

    上面提到了两种类型的锁：共享读锁（F_RDLCK）和独占写琐（F_WRLCK）。 
    基本规则是：多个进程在一个给定的字节 上可以有一把共享的读锁，但是在一个给定字节上的写锁则只能由一个进程独用。更进一步而言，如果在一个给定字节上已经有一把或多把读锁，则不能在该字节上再加写锁；如果在一个字节上已经有一把独占性的写锁，则不能再对它加任何读锁。在下图给出了这些规则。

    <img src=/images/Linux中fcntl函数介绍/读写锁权限.png>

    > 加读锁时，该描述符必须是读打开； 加写锁时，该描述符必须是写打开

2. cmd参数
<table>
    <thead>
    <tr>
        <th align="center" valign="middle" style="width: 150px;" >描述</th>
        <th align="center" valign="middle" >说明</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>cmd = F_GETLK</td>
        <td>判断对应的结构体中给定区间中是否有对应结构体中的锁：<br>
                如果写锁存在，就会阻止我fcntl函数进行，并且会替换我给的结构体指针中的结构体内容。返回当前锁的信息，并且在l_pid 处返回锁的持有者。<br>
                如果写锁不存在，则将 _flock.l_type改为F_UNLCK ，其余不变</td>
    </tr>
    <tr>
        <td>arg</td>
        <td>struct flock结构体</td>
    </tr>
    <tr>
        <td>返回值</td>
        <td>成功返回0，失败返回-1</td>
    </tr>
    <tr>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>cmd = F_SETLK</td>
        <td>F_SETLK相对于F_GETLK就要常用很多。其目的顾名思义就是对文件指定区域加锁/解锁。<br>
                当加锁不成功时，即其上面本来就有规则不允许的条件的锁时，就会出错返回，errno会设置为EACCES或EAGAIN</td>
    </tr>
    <tr>
        <td>arg</td>
        <td>struct flock结构体</td>
    </tr>
    <tr>
        <td>返回值</td>
        <td>成功返回0，失败返回-1</td>
    </tr>
    <tr>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>cmd = F_SETLKW</td>
        <td>F_SETLKW即F_SETLK的阻塞等待版，W即wait。<br>
                当fcntl 的请求不能满足时，就会进入休眠，当请求创建的锁可用时会被信号中断休眠，进程就会被唤醒</td>
    </tr>
    <tr>
        <td>arg</td>
        <td>struct flock结构体</td>
    </tr>
    <tr>
        <td>返回值</td>
        <td>成功返回0，失败返回-1</td>
    </tr>
    </tbody>
</table>

代码示例：
```C
//测试一把锁
pid_t lock_test(int fd,int type,off_t offset,int whence,off_t len)
{
    struct flock lock;
    lock.l_type = type; /*F_RDLCK or F_WRLCK*/
    lock.l_start = offset;
    lock.l_whence = whence; /*SEEK_SET,SEEK_CUR or SEEK_END*/
    lock.l_len = len;

    if(fcntl(fd,F_GETLK,&lock) < 0)
        err_sys("fcntl error");

    if(lock.l_type == F_UNLCK) /*false*/
        return(0);
    return (lock.l_pid); /*true*/
}


#define read_lock(fd,offset,whence,len)\
        lock_reg((fd),F_SETLK,F_RDLCK,(offset),(whence),(len))
#define readw_lock(fd,offset,whence,len)\
        lock_reg((fd),F_SETLKW,F_RDLCK,(offset),(whence),(len))
#define write_lock(fd,offset,whence,len)\
        lock_reg((fd),F_SETLK,F_WRLCK,(offset),(whence),(len))
#define writew_lock(fd,offset,whence,len)\
        lock_reg((fd),F_SETLKW,F_WRLCK,(offset),(whence),(len))
#define un_lock(fd,offset,whence,len)\
        lock_reg((fd),F_SETLK,F_UNLCK,(offset),(whence),(len))

//请求和释放一把锁
int lock_reg(int fd,int cmd,int type,off_t offset,int whence,off_t len)
{
    struct flock lock;
    lock.l_type = type;
    lock.l_start = offset;
    lock.l_whence = whence;
    lock.l_len = len;

    return (fcntl(fd,cmd,&lock));
}

```

### 锁区域

**在设置或释放文件上的一把锁时，系统按需组合或裂开相邻区**。例如，若对字节 0-99设置 一把读锁，然后对字节0-49设置一把写锁，则有两个加锁区： 0-49字节（写锁）及50-99（读锁）。又如，若100-199字节是加锁的区，需解锁第150字节，则内核将维持两把锁，一把用于100-149字节，另一把用于151-199字节

### 锁的继承和释放

关于记录锁的自动继承和释放有三条规则： 
1. **锁与进程、文件两方面有关**。这有两重含意：第一重很明显，当一个进程终止时，它所建立的锁全部释放；第二重意思就不很明显，任何时候关闭一个描述符时，则该进程通过这一描述符可以存访的文件上的任何一把锁都被释放。这就意味着 如果执行下列四步： 

    ```C
    fd1 = open(pathname, ...); 
    read_lock(fd1, ...); 
    fd2 = dup(fd1); 
    close(fd2); 
    ```

    则在`close(fd2)`后，在fd1上设置的锁被释放。如果将`dup`代换为`open`，其效果也一样：
    ```C 
    fd1 = open(pathname, ...); 
    read_lock(fd1, ...); 
    fd2 = open(pathname, ...);
    close(fd2); 
    ```

2.  **由fork产生的子程序不继承父进程所设置的锁**。这意味着，若一个进程得到一把锁，然后调用fork，那么对于父进程获得的锁而言，子进程被视为另一个进程，对于从父进程处继承过来的任一描述符，子进程要调用 fcntl以获得它自己的锁。这与锁的作用是相一致的。锁的作用是阻止多个进程同时写同一个文件（或同一文件区域）。如果子进程继承父进程的锁，则父、子进程就可以同时写同一个文件。 

3. **在执行exec后，新程序可以继承原执行程序的锁**。 
    > POSIX.1没有要求这一点。但是，SVR4和4.3+BSD都支持这一点。

### 文件尾端加锁

我们先来看一段代码

```C
writew_lock(fd,0,SEEK_END,0);//从内容末到文件末阻塞上读锁
write(fd,buf,1);
un_lock(fd,0,SEEK_END,0);//从内容末到文件末解锁
write(fd,buf,1);
```

看起来并没有什么问题，但其实上述中的两个内容末并不是同一位置(如下图所示)，我们在上锁后还进行了写入操作导致内容后移，所以当我们上文件末锁操作后还有解锁需求时，要记得相对偏移量。

<img src=/images/Linux中fcntl函数介绍/文件尾端加锁.png  width="50%">

---
