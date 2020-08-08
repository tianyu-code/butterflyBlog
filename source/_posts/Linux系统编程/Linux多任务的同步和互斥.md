---
title: Linux多任务的同步和互斥
date: 2020-08-06 14:36:32
tags:
  - Linux
  - 多任务的同步和互斥
categories:
  - Linux系统编程

keywords: Linux,系统编程,多任务的同步和互斥,互斥锁,mutex,信号量,sem,条件变量
cover: /images/封面图/p6.webp
top_img: /images/封面图/p6.webp
description: 本文介绍Linux多任务的同步和互斥机制，包括互斥锁，条件变量和信号量的使用。
sticky: 0

---


#  简介

**同步和互斥的区别和联系:**

1. 区别：
  **互斥：**是指散布在不同进程之间的若干程序片断，当某个进程运行其中一个程序片段时，其它进程就不能运行它们之中的任一程序片段，只能等到该进程运行完这个程序片段后才可以运行。
  **同步：**是指散布在不同进程之间的若干程序片断，它们的运行必须严格按照规定的 某种先后次序来运行，这种先后次序依赖于要完成的特定的任务。

2. 联系：
  `同步是一种更为复杂的互斥，而互斥是一种特殊的同步`。也就是说互斥是两个线程之间不可以同时运行，他们会相互排斥，必须等待一个线程运行完毕，另一个才能运行，而同步也是不能同时运行，但他是必须要安照某种次序来运行相应的线程（也是一种互斥）。

> 经常使用的方式有互斥锁、条件变量和信号量（有名和无名）。

# 互斥锁

## 简介
在多线程编程中，引入了对象互斥锁的概念，来保证共享数据操作的完整性。 每个对象都对应于一个可称为" 互斥锁" 的标记，这个标记用来保证在任一时刻， 只能有一个线程访问该对象。

> 若mutex的阻塞状态被信号中断，则信号处理返回后继续阻塞

## 函数

### pthread_mutex_init

```C
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
功能：该函数将使用attr参数初始化mutex，若attr传入null，则为默认值
成功返回0，失败返回错误码
```

**函数说明：**
1. 若函数调用成功，则mutex变为已初始化的并且是解锁状态
2. 只有已初始化的mutex本身能用于互斥，不能使用赋值来代替初始化。
3. 尝试初始化一个已经被初始化了的mutex的结果是未知的。
4. 可用`PTHREAD_MUTEX_INITIALIZER`宏来初始化一个静态分配的mutex，在这种情况下attr为null。


**attr的取值：**

+ **PTHREAD_MUTEX_NORMAL**：这是`缺省值`，也就是普通锁。当一个线程加锁以后，其余请求锁的线程将形成一个等待队列，并在解锁后按优先级获得锁。这种锁策略保证了资源分配的公平性。不应提供死锁检测。 同一线程尝试重新锁定mutex会导致死锁，若尝试解锁一个未被上锁或已经解锁的mutex，结果是未知的。

+ **PTHREAD_MUTEX_ERRORCHECK**：检错锁。提供`死锁检测`。同一线程尝试重新锁定mutex会返回错误，若尝试解锁一个未被上锁或已经解锁的mutex也会返回错误。其余行为和普通锁相同

+ **PTHREAD_MUTEX_RECURSIVE**：嵌套锁。提供`锁计数`的概念，每次上锁，计数值加1，每次解锁，计数值减1，当计数值变为0的时候，其余线程才能获得该锁。若尝试解锁一个未被上锁或已经解锁的mutex也会返回错误。

+ **PTHREAD_MUTEX_DEFAULT**：和`PTHREAD_MUTEX_NORMAL`一样。


### pthread_mutex_destroy
```C
int pthread_mutex_destroy(pthread_mutex_t *mutex);
功能：该函数销毁mutex，变为未初始化的，即无效值。
成功返回0，失败返回错误码
```
**函数说明：**
1. 可以使用`pthread_mutex_init`再次对已销毁mutex进行初始化，但是在被销毁后，其他方式的使用结果都是不确定的。
2. 销毁已经解锁的mutex是安全的，尝试销毁上锁的mutex会导致不确定的结果。

### pthread_mutex_lock
```C
int pthread_mutex_lock(pthread_mutex_t *mutex);
功能：给mutex上锁，若mutex已经上锁，其他线程想要上锁则阻塞直到其解锁
成功返回0，失败返回错误码
```

### pthread_mutex_trylock
```C
int pthread_mutex_trylock(pthread_mutex_t *mutex);
功能：给mutex上锁，若mutex已经上锁（本线程或其他线程），函数立即返回
若mutex可用则成功返回0，失败返回错误码
```

> 若锁类型为`PTHREAD_MUTEX_RECURSIVE`：嵌套锁，则本线程调用该函数时，锁计数加1，然后立即返回

### pthread_mutex_unlock
```C
int pthread_mutex_unlock(pthread_mutex_t *mutex);
功能：
成功返回0，失败返回错误码
```


# 条件变量

## 简介

与互斥锁不同，**条件变量是用来等待而不是用来上锁的**。条件变量用来自动阻塞一个线程，直到某特殊情况发生为止。



条件变量使我们可以睡眠等待某种条件出现。条件变量是利用线程间共享的全局变量进行**同步**的一种机制，主要包括两个动作：一个线程等待"条件变量的条件成立"而挂起；另一个线程使"条件成立"（给出条件成立信号）。

条件的检测是在互斥锁的保护下进行的。如果一个条件为假，一个线程自动阻塞，并释放等待状态改变的互斥锁。如果另一个线程改变了条件，它发信号给关联的条件变量，唤醒一个或多个等待它的线程，重新获得互斥锁，重新评价条件。

这个方法个人没有实际用到过，体会不是很深刻，查阅资料后个人理解条件变量主要解决的问题是：`当共享数据发生某个条件的变化时，才执行某些操作`。若没有这个机制，那么实现方式就可能是频繁的上锁解锁，频繁的判断，浪费时间，而引入这个机制后解决了这个问题

> 通常条件变量和互斥锁同时使用。

## 函数

### pthread_cond_init
```C
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
功能：该函数将使用attr参数初始化cond，若attr传入null，则为默认值
成功返回0，失败返回错误码
```
**说明：**
1. 只有已初始化的cond本身能用于同步，不能使用赋值来代替初始化。
2. 尝试初始化一个已经被初始化了的cond的结果是未知的。

### pthread_cond_destroy
```C
int pthread_cond_destroy(pthread_cond_t *cond);
功能：销毁条件变量
成功返回0，失败返回错误码
```

**说明：**
1. 可以使用pthread_cond_init再次对已销毁cond进行初始化，但是在被销毁后，其他方式的使用结果都是不确定的。
2. 销毁没有阻塞线程的cond是安全的，反之不安全

### pthread_cond_wait
```C
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
功能：等待条件变量
成功返回0，失败返回错误码
```
**说明：**

1. 在函数中会进行解锁`mutex`，阻塞在cond，等到cond满足后再上锁`mutex`的操作
2. 从`pthread_cond_wait`和`pthread_cond_timedwait`函数返回后都需要对条件进行重新判断，因为可能意外返回
3. 等待被信号中断时，从信号处理函数返回后，可能会继续等待，也可能会取消等待，且返回值为0（对应第二点来解决）


### pthread_cond_timedwait
```C
int pthread_cond_timedwait(pthread_cond_t *restrict cond,
              pthread_mutex_t *restrict mutex,
              const struct timespec *restrict abstime);
功能：等待条件变量，并设置超时返回
成功返回0，失败返回错误码
```
**说明：**

1. 这个函数除了有超时设置，其余操作和`pthread_cond_wait`无区别
2. 超时时间设置可如下（设置为5s）：
    ```C
    clock_gettime(CLOCK_REALTIME, &ts);
    ts.tv_sec += 5;
    ```

### pthread_cond_broadcast

```C
int pthread_cond_broadcast(pthread_cond_t *cond);
功能：唤醒所有被阻塞的线程
成功返回0，失败返回错误码
```
### pthread_cond_signal
```C
int pthread_cond_signal(pthread_cond_t *cond);
功能：唤醒一个被阻塞的线程
成功返回0，失败返回错误码
```

## 原理及示例代码

查阅资料后个人理解如下：

举个例子，就比方说生产者和消费者，那简单点就是店铺和顾客吧

```C
顾客的线程
{
    while(1)
    {
        printf("来了一位新顾客，要买一份炸鸡");
        pthread_mutex_lock(mutex);//敲敲玻璃叫老板

        printf("问问老板还有没有炸鸡");
        while(没有炸鸡了)
            {
            pthread_cond_wait(cond,mutex);//等着炸鸡
        }
        printf("顾客成功买到炸鸡了");
        pthread_mutex_unlock(&mutex);//离开店铺
    }
}


老板的线程
{
    while(1)
    {
        pthread_mutex_lock(mutex);//老板解决完其他事情了，可以安心做炸鸡了
        
        printf("老板一通操作，做炸鸡ing");
        if(有炸鸡了)
        {
            pthread_cond_signal（cond）；//告诉顾客有炸鸡了哈
        }
        pthread_mutex_unlock(&mutex);//老板累了，歇一会，干点别的事情，卖卖货啥的
    }
}


```

这是使用上述方法，假如单纯的使用`mutex`，效果如下

```C

顾客的线程
{
    while(1)
    {
        printf("来了一位新顾客，要买一份炸鸡");
        
        while(1)
        {
            pthread_mutex_lock(mutex);//敲敲玻璃叫老板
            printf("问问老板还有没有炸鸡");
            if(有炸鸡了)
            {
                printf("顾客成功买到炸鸡了");
                pthread_mutex_unlock(&mutex);
                break；//离开店铺
            }
            else
            {
                //没有炸鸡
                pthread_mutex_unlock(&mutex);
                sleep（1）；//歇会，接着等
            }
        }
    }
}

```
可以发现这种方法，要成功买到炸鸡，要问老板很多次，占用了老板很多时间，而老板回答你的问题的时候，是没法做饭的，这就导致了资源浪费。我们转换成代码看一下

```C

int x,y;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
                                                                                          
void *thread1(void *para)
{
    while(1)
    {
        pthread_mutex_lock(&mutex);
        while (x <= y) {
            pthread_cond_wait(&cond, &mutex);
        }
        /* 对 x、y 进行操作 */
        pthread_mutex_unlock(&mutex);
    }
}

void *thread2(void *para)
{
    while(1)
    {
        pthread_mutex_lock(&mut);
        /* 修改 x、y */
        if (x > y) 
        {
            pthread_cond_signal(&cond);
        }

        pthread_mutex_unlock(&mut);
    }
}

```
可能经过上述介绍发现，哎，这不是死锁了嘛？的确，从代码来看，先上锁等待其他线程条件满足，然后唤醒本线程，可是由于上锁了其他线程无法访问共享资源而导致死锁，其实，pthread_cond_wait中还进行了其他操作：
1. 线程放在等待队列上，解锁
2. 等待 pthread_cond_signal或者pthread_cond_broadcast信号之后去竞争锁
3. 若竞争到互斥索则加锁。


到这里就能知道，wait中会解锁，让其他线程操作共享资源，待条件满足后唤醒本线程，然后还要上锁，这样其他线程会安全的执行完，待其他线程解锁后，本线程才会执行后续操作。

这里还需要注意：对条件的判断需要使用`while`，也就是说线程从`pthread_cond_wait`返回后，需要再次主动判断，满足条件才可以继续执行，为了避免`虚假的唤醒`



# 信号量


## 简介

信号量广泛用于进程或线程间的同步和互斥，信号量本质上是一个非负的整数计数器，它被用来控制对公共资源的访问。

编程时可根据操作信号量值的结果判断是否对公共资源具有访问的权限，当信号量值大于 0 时，则可以访问，否则将阻塞。PV 原语是对信号量的操作，一次 P 操作使信号量减１，一次 V 操作使信号量加１。

在POSIX标准中，信号量分两种，一种是**无名信号量**，一种是**命名信号量**。无名信号量只用于线程间，命令信号量只用于进程间

**命令信号量和无名信号量的区别和联系**：除了创建和销毁的操作不同外，其余操作均相同。

## 函数总览


<table>
    <thead>
      <tr>
          <th align="center" valign="middle">函数</th>
          <th align="center" valign="middle">功能</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" colspan="2" bgcolor=#00a8f3>无名信号量</td>
      </tr>
      <tr>
          <td>sem_init</td>
          <td>初始化无名信号量</td>
      </tr>
      <tr>
          <td>sem_getvalue</td>
          <td>获取信号当前值</td>
      </tr>
      <tr>
          <td>sem_post</td>
          <td>指定的信号量的值加一</td>
      </tr>
      <tr>
          <td>sem_wait</td>
          <td>指定的信号量的值减一，若sem的值为0则阻塞直到可进行减操作</td>
      </tr>
      <tr>
          <td>sem_trywait</td>
          <td>指定的信号量的值减一，若sem的值为0则立即返回错误，但不阻塞</td>
      </tr>
      <tr>
          <td>sem_timedwait</td>
          <td>和wait相同，但指定阻塞超时时间，若超时退出则返回ETIMEDOUT</td>
      </tr>
      <tr>
          <td align="center" valign="middle" colspan="2" bgcolor=#00a8f3>命名信号量</td>
      </tr>
      <tr>
          <td>sem_open</td>
          <td>打开一个已存在的有名信号量，或创建并初始化一个有名信号量</td>
      </tr>
      <tr>
          <td>sem_close</td>
          <td>关闭有名信号量</td>
      </tr>
      <tr>
          <td>sem_unlink</td>
          <td>删除有名信号量</td>
      </tr>
    </tbody>
</table>

## 函数介绍


### sem_init
```C
int sem_init(sem_t *sem, int pshared, unsigned int value);
功能：初始化无名信号量
参数：
sem：待初始化的信号量
pshared：0代表在线程中共享
        非0代表在进程中共享，该信号量需要放到共享内存中以便其他进程访问（若是fork产生的子进程，也能正常使用该sem）
value:指定信号的初始值

成功返回0，失败返回-1

常见用法：sem_init(&sem, 0, 0);
```

### sem_getvalue
```C
int sem_getvalue(sem_t *sem, int *sval);
功能：获取信号当前值
成功返回0，失败返回-1
```
### sem_post
```C
int sem_post(sem_t *sem);
功能：指定的信号量的值加一
成功返回0，失败返回-1
```
### sem_wait
```C
int sem_wait(sem_t *sem);
功能：指定的信号量的值减一，若sem的值为0则阻塞直到可进行减操作
成功返回0，失败返回-1
```
> 注意：可被信号打断

### sem_trywait
```C
int sem_trywait(sem_t *sem);
功能：指定的信号量的值减一，若sem的值为0则立即返回错误，但不阻塞
成功返回0，失败返回-1
```

### sem_timedwait
```C
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
功能：和wait相同，但指定阻塞超时时间，若超时退出则返回ETIMEDOUT
成功返回0，失败返回-1

超时时间设置示例（5s）：
clock_gettime(CLOCK_REALTIME, &now);
now.tv_sec += 5;
```

### sem_open
```C
sem_t *sem_open(const char *name, int oflag);
sem_t *sem_open(const char *name, int oflag,
                       mode_t mode, unsigned int value);
功能：打开一个已存在的有名信号量，或创建并初始化一个有名信号量
参数：
name:文件名，不要带路径
oflag：O_CREAT或O_CREAT|EXCL
mode：控制新的信号量的访问权限
value:指定信号量的初始化值
返回值为sem
成功返回信号量地址，失败返回SEM_FAILED
```

**函数说明：**
1. 有名信号量默认创建在/dev/shm目录下
2. O_CREAT|EXCL代表若已存在则立刻返回错误


### sem_close
```C
int sem_close(sem_t *sem);
功能：关闭有名信号量
成功返回0，失败返回-1
```
### sem_unlink
```C
int sem_unlink(const char *name);
功能：删除有名信号量
成功返回0，失败返回
```
> 若信号量被打开（文件引用计数不为0），则删除操作无效


---