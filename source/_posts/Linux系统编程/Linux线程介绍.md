---
title: Linux线程介绍
date: 2020-08-01 17:36:32
tags:
  - Linux
  - 线程
categories:
  - Linux系统编程

keywords: Linux,系统编程,线程
cover: /images/封面图/CPU.jpg
top_img: /images/封面图/CPU.jpg
description: 本文介绍Linux中线程的概念，线程和进程的区别以及相关函数的使用方法。
sticky: 0

---


#  概念介绍

线程-调度的基本单位

## 线程和进程的区别
1. 进程是资源分配的基本单位
2. 创建一个线程的资源成本小，工作效率高
3. 线程是cpu或操作系统调度的基本单位

## 线程和进程共享的资源
+ 同一块地址空间
+ 文件描述符表
+ 信号处理方式
+ 当前工作目录
+ 用户id和组id

## 线程独立的资源
+ 栈空间
+ 线程ID
+ 一组寄存器的值
+ errno变量
+ 信号屏蔽字以及调度优先级

# 相关函数介绍

函数汇总：

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 300px;">函数</th>
          <th align="center" valign="middle" >功能</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>线程的创建及退出</td>
      </tr>   
      <tr>
          <td>pthread_create</td>
          <td>创建线程</td>
      </tr>
      <tr>
          <td>pthread_join</td>
          <td>等待线程终止</td>
      </tr>
      <tr>
          <td>pthread_detach</td>
          <td>将线程变为detach状态</td>
      </tr>
      <tr>
          <td>pthread_exit</td>
          <td>终止线程并返回终止状态</td>
      </tr>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>线程创建参数</td>
      </tr>   
      <tr>
          <td>phread_attr_init</td>
          <td>初始化attr参数</td>
      </tr>
      <tr>
          <td>pthread_attr_destroy</td>
          <td>销毁attr参数</td>
      </tr>
      <tr>
          <td>pthread_attr_setstack</td>
          <td>设置线程的栈地址和栈大小</td>
      </tr>
      <tr>
          <td>pthread_attr_setstacksize</td>
          <td>设置线程的栈大小</td>
      </tr>
      <tr>
          <td>pthread_attr_setdetachstate</td>
          <td>设置attr参数中的detach属性</td>
      </tr>
      <tr>
          <td>pthread_attr_setinheritsched</td>
          <td>设置attr的继承调度策略</td>
      </tr>
      <tr>
          <td>pthread_attr_setschedparam</td>
          <td>设置attr的调度参数</td>
      </tr>
      <tr>
          <td>pthread_attr_setschedpolicy</td>
          <td>设置attr的调度策略</td>
      </tr>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>线程取消及相关属性</td>
      </tr>   
      <tr>
          <td>pthread_setcancelstate</td>
          <td>设置本线程是否允许被cancel</td>
      </tr>
      <tr>
          <td>pthread_setcanceltype</td>
          <td>设置本线程的取消类型</td>
      </tr>
      <tr>
          <td>pthread_testcancel</td>
          <td>设置取消点</td>
      </tr>
      <tr>
          <td>pthread_cancel</td>
          <td>取消线程</td>
      </tr>
      <tr>
          <td>pthread_cleanup_push</td>
          <td>注册线程清理函数</td>
      </tr>
      <tr>
          <td>pthread_cleanup_pop</td>
          <td>执行（撤销）线程清理函数</td>
      </tr>
    </tbody>
</table>

## 线程创建

### pthread_create
```C
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
功能：创建线程。
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
          <td align="center" valign="middle" rowspan="4">参数</td>
          <td>thread</td>
          <td>返回线程ID</td>
          <td rowspan="5" >attr参数决定了创建的线程属性，由后续的phread_attr_init初始化，若填NULL则为默认属性</td>
      </tr>
      <tr>
          <td>attr</td>
          <td>设置线程属性</td>
      </tr>
      <tr>
          <td>start_routine</td>
          <td>线程处理函数</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>传递给线程函数的参数</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误值</td>
      </tr>
    </tbody>
</table>


## 线程属性

### phread_attr_init

```C
int pthread_attr_init(pthread_attr_t *attr);
功能：初始化attr参数
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
          <td align="center" valign="middle" >参数</td>
          <td>attr</td>
          <td>指定要初始化的参数</td>
          <td rowspan="2" >调用该函数对attr初始化后才可调用后续的函数来设置attr的属性值</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回非0</td>
      </tr>
    </tbody>
</table>

### pthread_attr_destroy

```C
int pthread_attr_destroy(pthread_attr_t *attr);
功能：销毁attr参数
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
          <td align="center" valign="middle" >参数</td>
          <td>attr</td>
          <td>指定要销毁的参数</td>
          <td rowspan="2" >1.若attr已经不再使用了，则应调用pthread_attr_destroy销毁，此操作对已经创建的线程无影响<br>2.已经销毁的attr可再次初始化，若后续函数使用已经销毁的attr，则会导致不确定的结果</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回非0</td>
      </tr>
    </tbody>
</table>

### pthread_attr_setstack

```C
int pthread_attr_setstack(pthread_attr_t *attr, void *stackaddr, size_t stacksize);
功能：设置线程的栈地址和栈大小
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
          <td align="center" valign="middle" rowspan="3" >参数</td>
          <td>attr</td>
          <td>已初始化的attr参数</td>
          <td rowspan="4" >注意：该函数是为了那些需要将栈放在指定位置的线程提供的，默认情况下应避免使用该函数</td>
      </tr>
      <tr>
          <td>stackaddr</td>
          <td>指向栈空间的最低字节</td>
      </tr>
      <tr>
          <td>stacksize</td>
          <td>指定了栈空间的大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_attr_setstacksize

```C
int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
功能：设置线程的栈大小
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
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>attr</td>
          <td>已初始化的attr参数</td>
          <td rowspan="3" >无</td>
      </tr>
      <tr>
          <td>stacksize</td>
          <td>指定了栈空间的大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_attr_setdetachstate

```C
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
功能：设置attr参数中的detach属性
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
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>attr</td>
          <td>已初始化的attr参数</td>
          <td rowspan="3" >不调用该函数，默认为joinable</td>
      </tr>
      <tr>
          <td>detachstate</td>
          <td>PTHREAD_CREATE_DETACHED：线程属性是detached的<br>PTHREAD_CREATE_JOINABLE：线程属性是joinable的</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_attr_setinheritsched

```C
int pthread_attr_setinheritsched(pthread_attr_t *attr, int inheritsched);
功能：设置attr的继承调度策略
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
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>attr</td>
          <td>已初始化的attr参数</td>
          <td rowspan="3" >继承调度策略决定了线程的调度策略是继承线程创建者，还是attr中指定的</td>
      </tr>
      <tr>
          <td>inheritsched</td>
          <td>PTHREAD_INHERIT_SCHED：继承线程创建者的调度策略，attr参数中的相关属性将被忽略<br>PTHREAD_EXPLICIT_SCHED：使用attr参数中指定的调度策略</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_attr_setschedparam

```C
int pthread_attr_setschedparam(pthread_attr_t *attr, const struct sched_param *param);
功能：设置attr的调度参数
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
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>attr</td>
          <td>已初始化的attr参数</td>
          <td rowspan="3" >该函数设置了attr中，关于线程调度的参数，该参数如下：<br>
           struct sched_param {<br>
               int sched_priority;     /* Scheduling priority */<br>
           };<br>即调度的优先级，系统支持的最大和最小的优先级值可以用函数：<br>sched_get_priority_max和sched_get_priority_min得到</td>
      </tr>
      <tr>
          <td>param</td>
          <td>调度参数</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>


### pthread_attr_setschedpolicy
```C
int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
功能：设置attr的调度策略
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
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>attr</td>
          <td>已初始化的attr参数</td>
          <td rowspan="3" >该函数设置的调度策略决定了使用attr参数创建的线程的调度策略，如policy所示，前两种均需要设置调度的优先级，数值越大优先级越高，且这两种方式均支持高优先级抢占，而RR不同的是设置了最大运行的时间片。<br>而SCHED_OTHER的优先级永远是0，它是标准的Linux分时调度程序，适用于不需要特殊实时机制的所有线程。</td>
      </tr>
      <tr>
          <td>policy</td>
          <td>调度策略：<br>SCHED_FIFO：先入先出<br>SCHED_RR：轮转法<br>SCHED_OTHER：其他方法</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

## 线程退出

### pthread_join
```C
int pthread_join(pthread_t thread, void **retval);
功能：等待线程终止
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
          <td>thread</td>
          <td>等待的线程ID</td>
          <td rowspan="3" >1.若线程已经终止，则该函数立刻返回<br>2.若retval不是NULL，则将线程退出状态通过该参数返回，若线程是被cancel的，则返回PTHREAD_CANCELED<br>注意：这里的retval使用二级指针，是因为pthread_exit返回的终止状态是一个指针，而我们要拿到这个指针就必须传入一个指针的地址。</td>
      </tr>
      <tr>
          <td>retval</td>
          <td>线程退出状态</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误值</td>
      </tr>
    </tbody>
</table>


### 线程终止的情况

1. 调用pthread_exit，且将线程退出状态通过参数传递给pthread_join
2. 从线程函数return，效果同上
3. 线程被取消
4. 进程中的任意线程调用exit，或者主线程返回，都会导致所有线程终止

### pthread_detach
```C
int pthread_detach(pthread_t thread);
功能：将线程变为detach状态
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
          <td align="center" valign="middle" >参数</td>
          <td>thread</td>
          <td>指定线程ID</td>
          <td rowspan="2" >线程detach后，则由系统自动回收线程资源，无需主动join</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误值</td>
      </tr>
    </tbody>
</table>

### joinable和detached状态
1. 若是joinable的，则其他线程可调用pthread_join来等待该线程终止并获取退出状态，且必须通过该方式来回收线程资源
2. 若是detached的，则无需主动pthread_join,线程退出后资源会被自动回收，且无法获取线程退出状态
3. 默认创建的线程是joinable的，除非主动设置attr（pthread_attr_setdetachstate）

### pthread_exit
```C
void pthread_exit(void *retval);
功能：终止线程并返回终止状态
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
          <td align="center" valign="middle" >参数</td>
          <td>retval</td>
          <td>函数返回状态</td>
          <td rowspan="2" >1.通过*retval将终止状态返回给pthread_join<br>2.执行所有注册的线程清理函数</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>void</td>
          <td>无</td>
      </tr>
    </tbody>
</table>


## 线程取消


### pthread_setcancelstate

```C
int pthread_setcancelstate(int state, int *oldstate);
功能：设置本线程是否允许被cancel
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
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>state</td>
          <td>可被取消属性：<br>PTHREAD_CANCEL_ENABLE：可被取消<br>PTHREAD_CANCEL_DISABLE：不可被取消</td>
          <td rowspan="3" >默认是可被取消的</td>
      </tr>
      <tr>
          <td>oldstate</td>
          <td>旧的属性将被从该参数返回</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_setcanceltype

```C
int pthread_setcanceltype(int type, int *oldtype);
功能：设置本线程的取消类型
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 450px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="2" >参数</td>
          <td>type</td>
          <td>取消类型：<br>PTHREAD_CANCEL_DEFERRED：延迟取消，直到遇到取消点（若取消点在信号处理函数中也可生效，这样效果就和异步取消很相似了）<br>
                            PTHREAD_CANCEL_ASYNCHRONOUS：异步取消，线程可以随时取消（通常在收到取消命令后立刻取消，但系统不对此保证）</td>
          <td rowspan="3" >默认类型是PTHREAD_CANCEL_DEFERRED</td>
      </tr>
      <tr>
          <td>oldtype</td>
          <td>旧的属性将被从该参数返回</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_testcancel

```C
void pthread_testcancel(void);
功能：设置取消点
说明：在PTHREAD_CANCEL_DEFERRED模式下，当线程执行到该函数时，则可相应取消，若该线程不可取消或者无取消请求，则该函数无作用
```

### pthread_cancel

```C
int pthread_cancel(pthread_t thread);
功能：取消线程
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
          <td>thread</td>
          <td>取消的线程ID</td>
          <td rowspan="2" >1.实测cancel可使正在阻塞的线程强制退出<br>
                            2.若目标线程是不允许被cancel的，那么该函数发出的cancel请求会被遗留直到目标线程可被cancel再继续响应</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回错误码</td>
      </tr>
    </tbody>
</table>

### pthread_cleanup_push

```C
void pthread_cleanup_push(void (*routine)(void *), void *arg);
功能：注册线程清理函数
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
          <td>routine</td>
          <td>线程清理函数</td>
          <td rowspan="3" >注册线程的退出函数，当以下情况时，调用处理函数：<br>
                        （1）线程被取消<br>
                        （2）调用pthread_exit<br>
                        （3）调用pthread_cleanup_pop，且参数不为0，此时只会执行顶层的一个清理函数<br>
                        注意：从线程中return不用调用处理函数<br>
</td>
      </tr>
      <tr>
          <td>arg</td>
          <td>线程清理函数的参数</td>
      </tr>      
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>void</td>
          <td>无</td>
      </tr>
    </tbody>
</table>


### pthread_cleanup_pop

```C
void pthread_cleanup_pop(int execute);
功能：执行（撤销）线程清理函数
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
          <td>execute</td>
          <td>为0时，撤销顶层线程清理函数<br>非0时，执行顶层线程清理函数</td>
          <td rowspan="2" >1.实测cancel可使正在阻塞的线程强制退出<br>
                            2.若目标线程是不允许被cancel的，那么该函数发出的cancel请求会被遗留直到目标线程可被cancel再继续响应</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>void</td>
          <td>无</td>
      </tr>
    </tbody>
</table>

> 注意，这两个函数的实现是两个宏，pthread_cleanup_push包含了“{”,pthread_cleanup_pop包含了“}”，所以这两个函数必须成对出现