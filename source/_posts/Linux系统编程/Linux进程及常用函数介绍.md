---
title: Linux进程及常用函数介绍
date: 2020-08-01 11:36:32
tags:
  - Linux
  - 进程
categories:
  - Linux系统编程

keywords: Linux,系统编程,进程
cover: /images/封面图/Linux.jfif
top_img: /images/封面图/Linux.jfif
description: 本文介绍Linux中进程的概念以及相关函数的使用方法。
sticky: 0

---


#  概念介绍

## 进程-资源分配的基本单位

进程：通俗的讲就是一个程序运行的实例，是具有一定独立功能的程序在一个数据集合上的一次动态执行过程，它是动态的，包括创建，调度，执行和消亡。

## 特殊进程含义

名称 | 含义 |  
--- | --- |  
僵尸进程 | 子进程结束，父进程没有调用wait来回收资源 | 
孤儿进程 | 父进程结束，子进程还在运行 | 
守护进程（精灵进程） | 后台运行的孤儿进程 |

孤儿进程的取消方法：因为父进程已经结束了，所在的进程组就没有了，接收不到终端发过来的结束信号，所以要用ps，然后用kill杀掉

> `ps -A -ostat,ppid,pid,cmd | grep -e '^[Zz]'`这条命令可以查看僵尸进程

## 进程组的概念

一个进程会有如下 ID：

1. PID：进程的唯一标识。如果一个进程含有多个线程，所有线程调用 getpid 函数会返回相同的值。
2. PGID：进程组 ID。每个进程都会有进程组 ID，表示该进程所属的进程组。默认情况下新创建的进程会继承父进程的进程组 ID。
3. SID：会话 ID。每个进程也都有会话 ID。默认情况下，新创建的进程会继承父进程的会话 ID。
4. PPID：是程序的父进程号

进程组和会话在进程之间形成了两级的层次关系，`ps axjf`可查看.

《Linux环境编程：从应用到内核》这本书里，有一个解释，说的比较生动形象

1. 可以打个比方，以家族企业的创业为例，每个进程可以比喻成家族企业的每个成员。
2. 如果从创业之初，所有家族成员都安分守己，循规蹈矩，默认情况下，就只会有一个公司、一个部门。但是也有些“叛逆”的子弟，愿意为家族公司开疆拓土，愿意成立新的部门。
3. 这些新的部门就是新创建的进程组。如果有子弟“离经叛道”，甚至不愿意呆在家族公司里，他别开天地，另创了一个公司，那这个新公司就是新创建的会话组。由此可见，系统必须要有改变和设置进程组 ID 和会话 ID 的函数接口，否则，系统中只会存在一个会话、一个进程组

## 知识点总结

1. 子进程正常退出后，会变成僵尸进程，剩余一些信息以供父进程获取，父进程必须使用wait或者waitpid来清理子进程

2. 子进程结束时，会向父进程发送SIGCHLD信号，以便父进程回收资源

3. 父进程退出时，若有未回收的资源，那么init进程会代为回收

4. 父进程退出时，若有运行中的子进程，它们会被init进程领养（即父进程改为1）

5. 父进程退出时，可使用prctl来通知子进程退出，即向子进程发送信号，例如prctl(PR_SET_PDEATHSIG,SIGKILL)

6. fork之后父子进程的执行顺序是不确定的，这取决于内核的调度算法

7. fork之后父子进程的缓冲区和文件位移量也会被复制


# 进程相关函数

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
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>创建进程</td>
      </tr>   
      <tr>
          <td>fork</td>
          <td>创建子进程，父子进程同时运行</td>
      </tr>
      <tr>
          <td>vfork</td>
          <td>创建子进程，子进程先运行</td>
      </tr>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>获取进程ID</td>
      </tr>   
      <tr>
          <td>getpid</td>
          <td>获取本进程ID</td>
      </tr>
      <tr>
          <td>getppid</td>
          <td>获取父进程ID</td>
      </tr>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>等待子进程</td>
      </tr>   
      <tr>
          <td>wait</td>
          <td>等待任意子进程</td>
      </tr>
      <tr>
          <td>waitpid</td>
          <td>等待指定pid的子进程</td>
      </tr>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>注册终止函数</td>
      </tr>   
      <tr>
          <td>atexit</td>
          <td>注册终止函数</td>
      </tr>
      <tr>
          <td colspan="2" align="center" valign="middle" bgcolor=#00a8f3>system函数</td>
      </tr>   
      <tr>
          <td>system</td>
          <td>注执行command命令册终止函数</td>
      </tr>
    </tbody>
</table>

函数详细介绍请见下文：


## 创建进程

1. `pid_t fork(void);`
功能：创建子进程
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
          <td>void</td>
          <td>无</td>
          <td rowspan="2" >1.fork复制父进程的各种信息，包括缓冲区和文件偏移量等<br>2.父子进程同时运行</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>pid_t</td>
          <td>对于父进程返回子进程ID，对于子进程返回0</td>
      </tr>
    </tbody>
</table>

2. `pid_t vfork(void);`
功能：创建子进程

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
          <td>void</td>
          <td>无</td>
          <td rowspan="2" >1.与父进程共用堆栈，用vfork一般都会调用exec来重新加载代码段和数据段<br>2.保证子进程先运行等待子进程调用exit或者exec之后父进程才可能被调度运行（如果在调用这两个函数之前子进程依赖于父进程的进一步动作，则会导致死锁）<br>3.注意用vfork创建子进程中，不能用return结束子进程，因为父子进程公用堆栈，return会弹栈，让程序认为父进程已经结束，会有出乎意料的后果，所以要用exit来结束</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>pid_t</td>
          <td>对于父进程返回子进程ID，对于子进程返回0</td>
      </tr>
    </tbody>
</table>

## 获取进程ID

1. `pid_t getpid(void);`

  获取本进程ID，返回值即为结果。
2. `pid_t getppid(void);`

  获取父进程ID，返回值即为结果。

## 等待子进程

1. `pid_t wait(int *wstatus);`
功能：等待任意子进程
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
          <td>wstatus</td>
          <td>该参数返回子进程的终止状态，可用宏查看</td>
          <td rowspan="2" >1.只要有一个子进程结束了，就会立刻返回，继续运行程序了，如果一个子进程已经终止了，是僵尸进程，那么wait立即返回<br>2.使用宏查看子进程返回状态<br>(1)WIFEXITED(status)为真表示子进程正常返回<br>(2)WEXITSTATUS(status) 当WIFEXITED返回非零值时，我们可以用这个宏来提取子进程的返回值，例如子进程调用exit(5)退出，WEXITSTATUS(status) 就会返回5<br><br>（3）WIFSIGNALED(status) 为真表示子进程异常返回<br>（4）WTERMSIG(status) 取得子进程因信号而中止的信号代码,一般会先用 WIFSIGNALED 来判断后才使用此宏<br><br>（5）WIFSTOPPED(status) 为真表示当前子进程被在暂停<br>（6）WSTOPSIG(status) 取得引发子进程暂停的信号代码,一般会先用 WIFSTOPPED 来判断后才使用此宏</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>pid_t</td>
          <td>已终止的子进程PID</td>
      </tr>
    </tbody>
</table>

2. `pid_t waitpid(pid_t pid, int *wstatus, int options);`
功能：等待指定pid的子进程

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 470px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>pid</td>
          <td>pid参数<-1,等待进程组ID和pid绝对值相同的任意子进程<br>pid参数=-1，等待任意子进程<br>pid参数=0，等待本进程组内的任意子进程<br>pid>0，等待指定pid的子进程</td>
          <td rowspan="4" >1.waitpid(-1, &status, 0)和wait功能一样<br>2.status部分和wait函数相同</td>
      </tr>
      <tr>
          <td>wstatus</td>
          <td>该参数返回子进程的终止状态，可用宏查看</td>
      </tr>
      <tr>
          <td>options</td>
          <td>option参数为WNOHANG，若等待的子进程未结束则立刻返回，无阻塞<br>option参数为WUNTRACED，若子进程进入暂停状态，则马上返回，但子进程的结束状态不予以理会</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>pid_t</td>
          <td>已终止的子进程PID</td>
      </tr>
    </tbody>
</table>

## 注册进程终止函数

`int atexit(void (*function)(void));`

功能：注册进程终止函数

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
          <td>function</td>
          <td>注册的函数</td>
          <td rowspan="2" >1.在进程正常退出时，所有注册的终止函数将被执行，注意：调用_exit和_Exit函数退出的不执行清理动作，也就不会执行终止函数<br>2.可注册最多32个终止函数，后注册的先执行</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回非0</td>
      </tr>
    </tbody>
</table>

## system函数

`int system(const char *command);`

功能：该函数使用fork创建一个子进程,该子进程使用execl执行指定的shell命令

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
          <td>command</td>
          <td>被执行的命令</td>
          <td rowspan="2" >执行期间，阻止SIGCHLD信号，并且忽略SIGINT和SIGQUIT信号</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>1.若command为NULL，返回非0值<br>2.若创建子进程失败返回-1<br>3.创建的子进程的状态通过返回值返回，可参考wait函数对子进程返回状态的处理来判断是否成功</td>
      </tr>
    </tbody>
</table>



# exec函数族介绍

1. 系统调用exec是以新的进程去代替原来的进程，但进程的PID保持不变。因此，可以这样认为，exec系统调用并没有创建新的进程，只是替换了原来进程上下文的内容。原进程的代码段，数据段，堆栈段被新的进程所代替
2. exec中的函数执行成功后不会返回，只有失败了才会返回-1，失败后接着从源程序的地方往下执行
3. 常与vfork一起使用


函数原型如下：

```C
int execl(const char *pathname, const char *arg, ... /* (char  *) NULL */);
int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
int execle(const char *pathname, const char *arg, ... /*, (char *) NULL, char *const envp[] */);

int execv(const char *pathname, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[], char *const envp[]);
int execve(const char *pathname, char *const argv[], char *const envp[]);
```

函数 | 特点 |  
--- | --- |  
execl | 基础函数，提供程序路径和参数，参数通过arg分别传入，以NULL结尾 | 
execlp | 较execl相比，提供的程序可不带路径，而自动到PATH环境变量下查询 | 
execle | 较execl相比，提供的程序可不带路径，而自动到PATH环境变量下查询，调用本函数时，可使用envp参数设置环境变量，若不填写envp参数，则继承父进程的环境变量，envp也以NULL结尾 |
execv |  较execl相比，参数通过argv[]数组参入，数组中的参数还是以NULL结尾| 
execvp | 较execlp相比，参数通过argv[]数组参入，数组中的参数还是以NULL结尾 | 
execvpe | 较execle相比，参数通过argv[]数组参入，数组中的参数还是以NULL结尾 |
execve | 较execv相比，可提供环境变量（该函数需要提供可执行文件全路径）|








