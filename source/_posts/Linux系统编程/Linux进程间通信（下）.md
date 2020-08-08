---
title: Linux进程间通信（下）
date: 2020-08-04 15:36:32
tags:
  - Linux
  - 进程间通信
  - 消息队列、共享内存和信号量
categories:
  - Linux系统编程

keywords: Linux,系统编程,进程间通信,消息队列,共享内存
cover: /images/封面图/p4.webp
top_img: /images/封面图/p4.webp
description: 本文介绍Linux进程间通信中消息队列、共享内存和信号量的使用。
sticky: 0

---

其他进程间通信可参考[Linux信号](http://tianyu-code.top/Linux%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A8%8B/Linux%E4%BF%A1%E5%8F%B7/)和[Linux进程间通信（上）](http://tianyu-code.top/Linux%E7%B3%BB%E7%BB%9F%E7%BC%96%E7%A8%8B/Linux%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1%EF%BC%88%E4%B8%8A%EF%BC%89/)

本文介绍Linux进程间通信的消息队列，信号量和共享内存。

> 这三种方式均为System V 中的通信方式，切勿将此处的信号量与POSIX的信号量相混淆

#  消息队列

消息队列是消息的链接表 ,存放在内核中并由消息队列标识符标识。我们将称消息队列为“队列”，其标识符为“队列ID”

## 特点

1. 消息不一定要以先进先出的次序读取，编程时可以按消息的类型读取
2. 消息被读取后删除（和管道相同）
3. 消息队列有唯一标识符
4. 只有内核重启或手动删除才可以删除消息队列

## 预设值

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 100px;" bgcolor=#00a8f3>名称</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>说明</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>典型值</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>MSGMAX</td>
          <td>可发送的最长消息的字节长度</td>
          <td>2048</td>
      </tr>
      <tr>
          <td>MSGMNB</td>
          <td>特定队列的最大字节长度 (亦即队列中所有消息之和 )</td>
          <td>4096</td>
      </tr>
      <tr>
          <td>MSGMNI</td>
          <td>系统中最大消息队列数 </td>
          <td>50</td>
      </tr>
      <tr>
          <td>MSGTOL</td>
          <td>系统中最大消息数 </td>
          <td>50</td>
      </tr>    
    </tbody>
</table>

## 函数

### ftok
```C
key_t ftok(const char *pathname, int proj_id);
功能：获取唯一键值
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
          <td>pathname</td>
          <td>任意路径名（必须已存在）</td>
          <td rowspan="3" >该函数通过pathname和proj_id生成一个键值供后续System V的IPC通信使用（msgget, semget, or shmget）</td>
      </tr>
      <tr>
          <td>proj_id</td>
          <td>1-255</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>key_t</td>
          <td>成功返回消息队列的标识符，失败返回-1</td>
      </tr>
    </tbody>
</table>


### msgget
```C
int msgget(key_t key, int msgflg);
功能：创建一个新的或者打开一个已经存在的消息队列，只要有相同的key值就可以获得相同的消息队列
```
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 320px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="2">参数</td>
          <td>key</td>
          <td>ftok函数返回的键值</td>
          <td rowspan="3" >举例：<br>
                          int id = msgget(key,IPC_CREAT|IPC_EXCL|0666);<br>
                          创建一个权限为0666的消息队列，并返回一个整形消息队列ID，如果key值已经存在有消息队列了，则出错返回-1。<br>
                          <br>
                          int id = msgget(key,IPC_CREAT|0666);<br>
                          创建一个权限为0666的消息队列，并返回一个消息队列ID，如果key值已经存在有消息队列了，则直接返回一个消息队列ID。</td>
      </tr>
      <tr>
          <td>msgflg</td>
          <td>IPC_CREAT创建消息队列，且后面需要跟着权限<br>
              IPC_EXCL检查消息队列是否存在</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回消息队列的标识符，失败返回-1</td>
      </tr>
    </tbody>
</table>

### msgsnd
```C
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
功能:将新消息添加到消息队列
```
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 600px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="4">参数</td>
          <td>msqid</td>
          <td>消息队列的标识符</td>
          <td rowspan="5" >无</td>
      </tr>
      <tr>
          <td>msgp</td>
          <td>待发送消息结构体的地址</td>
      </tr>
      <tr>
          <td>msgsz</td>
          <td>消息正文的字节数</td>
      </tr>
      <tr>
          <td>msgflg</td>
          <td>函数的控制属性,如下所示：<br>
                  0：当消息队列满时，msgsnd将会阻塞，直到消息能写进消息队列或者消息队列被删除<br>
                  IPC_NOWAIT: 当消息队列满了，msgsnd函数将不会等待，会立即出错返回EAGAIN</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0；错误返回-1</td>
      </tr>
    </tbody>
</table>


### msgrcv
```C
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
功能：从标识符为msqid的消息队列中接收一个消息。
```
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 500px;">说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="5">参数</td>
          <td>msqid</td>
          <td>消息队列的标识符</td>
          <td rowspan="6" >若消息队列中有多种类型的消息，msgrcv获取消息的时候按消息类型获取，若队列中有多条此类型的消息，则获取最先添加的消息，即先进先出原则。一旦接收消息成功，则消息在消息队列中被删除。</td>
      </tr>
      <tr>
          <td>msgp</td>
          <td>存放消息结构体的地址</td>
      </tr>
      <tr>
          <td>msgsz</td>
          <td>消息正文的字节数</td>
      </tr>
      <tr>
          <td>msgtyp</td>
          <td>msgtyp == 0 返回队列中的第一个消息<br>
              msgtyp > 0 返回队列中消息类型为type的第一个消息<br>
              msgtyp < 0 返回队列中消息类型值小于等于type绝对值的消息，如果这种消息有若干个，则取类型值最小的消息</td>
      </tr>
      <tr>
          <td>msgflg</td>
          <td>函数的控制属性:<br>
            0：msgrcv调用阻塞直到接收消息成功为止。<br>
            MSG_NOERROR:若返回的消息字节数比nbytes字节数多,则消息就会截短到nbytes 字节,且不通知消息发送进程，若未指定则此时消息不会从队列中移除，且函数调用返回-1。<br>
            IPC_NOWAIT:调用进程会立即返回。若没有收到消息则立即返回-1。<br>
            IPC_EXCEPT时，与msgtype配合使用返回队列中第一个类型不为msgtype的消息</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>ssize_t</td>
          <td>成功返回消息数据部分的长度；错误返回-1</td>
      </tr>
    </tbody>
</table>

### msgctl
```C
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
功能:消息队列控制函数
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>msqid</td>
          <td>消息队列的标识符</td>
          <td rowspan="4" >对消息队列进行各种控制，如修改消息队列的属性，或删除消息消息队列。删除消息队列时，这种删除立即生效，仍在使用这一队列的其他进程在下一次试图对该队列操作时会返回EIDRM。<br>常见用法：msgctl(id,IPC_RMID,NULL);删除id号的消息队列</td>
      </tr>
      <tr>
          <td>cmd</td>
          <td>取值如下:<br>
            IPC_RMID：删除由msqid指示的消息队列，将它从系统中删除并破坏相关数据结构。<br>
            IPC_STAT：将msqid相关的数据结构中各个元素的当前值存入到由buf指向的结构中。<br>
            IPC_SET：将msqid相关的数据结构中的元素设置为由buf指向的结构中的对应值</td>
      </tr>
      <tr>
          <td>buf</td>
          <td>msqid_ds数据类型的地址，用来存放或更改消息队列的属性</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0；错误返回-1</td>
      </tr>
    </tbody>
</table>

### shell操作消息队列
```C
ipcs   -q 查看消息队列
ipcrm  -q  msgid  删除消息队列
```


## 示例代码

```C
struct msgbuf {
   long mtype;
   char mtext[80];
};



void send_msg(int qid, int msgtype)
{
   struct msgbuf msg;
   time_t t;

   msg.mtype = msgtype;

   time(&t);
   snprintf(msg.mtext, sizeof(msg.mtext), "a message at %s",
		   ctime(&t));

   if (msgsnd(qid, (void *) &msg, sizeof(msg.mtext),
			   IPC_NOWAIT) == -1) {
	   perror("msgsnd error");
	   exit(EXIT_FAILURE);
   }
   printf("sent: %s\n", msg.mtext);
}

void get_msg(int qid, int msgtype)
{
	struct msgbuf msg;

	if (msgrcv(qid, (void *) &msg, sizeof(msg.mtext), msgtype,
			  MSG_NOERROR | IPC_NOWAIT) == -1) {
	   if (errno != ENOMSG) {
		   perror("msgrcv");
		   exit(EXIT_FAILURE);
	   }
	   printf("No message available for msgrcv()\n");
	} else
	   printf("message received: %s\n", msg.mtext);
}

int main(int argc, char *argv[])
{
	int qid;
	int mode = 0;               /* 1 = send, 2 = receive */
	int msgtype = 1;
	int msgkey = 123;

	qid = msgget(msgkey, IPC_CREAT | 0666);

	if (qid == -1) {
	   perror("msgget");
	   exit(EXIT_FAILURE);
	}
	get_msg(qid, msgtype);

	send_msg(qid, msgtype);

	exit(EXIT_SUCCESS);
}


```


# 信号量

> 注意：这里的信号量是System V中的，不要和POSIX中的信号量搞混

## 简介

信号量与已经介绍过的IPC机构（管道、FIFO以及消息列队）不同。它是一个计数器，用于多进程对共享数据对象的存取。为了获得共享资源，进程需要执行下列操作： 
1. 测试控制该资源的信号量。 
2. 若此信号量的值为正，则进程可以使用该资源。进程将信号量值减 1，表示它使用了一个资源单位。 
3. 若此信号量的值为 0，则进程进入睡眠状态，直至信号量值大于 0。若进程被唤醒后，它返回至(第( 1 )步)。 
4. 当进程不再使用由一个信息量控制的共享资源时，该信号量值增 1。如果有进程正在睡眠等待此信号量，则唤醒它们。

> 为了正确地实现信息量，信号量值的测试及减1操作应当是原子操作。为此，信号量通常是在内核中实现的。 

常用的信号量形式被称之为双态信号量(binary semaphore)。它控制单个资源，其初始值为1。但是，一般而言，信号量的初值可以是任一正值，该值说明有多少个共享资源单位可供共享应用。 不幸的是，系统V的信号量与此相比要复杂得多。三种特性造成了这种并非必要的复杂性：

+ 信号量并非是一个非负值，而必需将信号量定义为含有一个或多个信号量值的集合。 当创建一个信号量时，要指定该集合中的各个值 
+ 创建信息量（semget）与对其赋初值（semctl）分开。这是一个致命的弱点，因为不能原子地创建一个信号量集合，并且对该集合中的所有值赋初值。 
+ 即使没有进程正在使用各种形式的系统V IPC，它们仍然是存在的，所以不得不为这种程序担心，它在终止时并没有释放已经分配给它的信号量。


## 预设值


<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 100px;" bgcolor=#00a8f3>名称</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>说明</th>
          <th align="center" valign="middle" style="width: 300px;" bgcolor=#00a8f3>典型值</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>SEMVM</td>
          <td>任一信号量的最大值</td>
          <td>32767</td>
      </tr>
      <tr>
          <td>SEMAE</td>
          <td>任一信号量的最大终止时调整值</td>
          <td>16384</td>
      </tr>
      <tr>
          <td>SEMMN</td>
          <td>系统中信号量集的最大数 </td>
          <td>10</td>
      </tr>
      <tr>
          <td>SEMMNS</td>
          <td>系统中信号量集的最大数 </td>
          <td>60</td>
      </tr>  
      <tr>
          <td>SEMMSL</td>
          <td>每个信号量集中的最大信号量数 </td>
          <td>25</td>
      </tr>   
    </tbody>
</table>

## 函数


### semget

```C
int semget(key_t key, int nsems, int semflg);
功能：创建一个新的或打开一个已经存在的信号量
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>key</td>
          <td>ftoke返回的键值</td>
          <td rowspan="4" >无</td>
      </tr>
      <tr>
          <td>nsems</td>
          <td>该集合中的信号量数。如果是创建新集合，则必须指定（一般为1）。如果引用一个现存的集合，则将其指定为0</td>
      </tr>
      <tr>
          <td>semflg</td>
          <td>IPC_CREAT创建消息队列，且后面需要跟着权限<br>
                IPC_EXCL检查消息队列是否存在</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回信号量标识，失败返回-1</td>
      </tr>
    </tbody>
</table>

### semop
```C
int semop(int semid, struct sembuf *sops, size_t nsops);
功能：修改信号量的值
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>semid</td>
          <td>信号量标识，由semget返回</td>
          <td rowspan="4" >struct sembuf{ <br> 
                            short sem_num; //除非使用一组信号量，否则它为0<br>  
                            short sem_op;//信号量操作，-1，即P（等待）操作，+1，即V（发送信号）操作。<br>
                            short sem_flg;<br>
                            //通常为SEM_UNDO,使操作系统跟踪信号，并在进程没有释放该信号量而终止时，操作系统释放信号量<br>  
                        }; </td>
      </tr>
      <tr>
          <td>sops</td>
          <td>信号量操作数组</td>
      </tr>
      <tr>
          <td>nsops</td>
          <td>信号量操作数组的数量</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>

### semctl 
```C
int semctl(int semid, int semnum, int cmd, ...)
功能：信号量控制函数
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>semid</td>
          <td>信号量标识，由semget返回</td>
          <td rowspan="4" >主要用于信号量的初始化和删除，若初始化，则后面还需要加入一个参数，为共用体：<br>
                            union semun{  <br>
                            int val; <br>
                            struct semid_ds *buf; <br> 
                            unsigned short *arry; <br> 
                            }; <br>
                            一般用到的是val,表示要传给信号量的初始值。</td>
      </tr>
      <tr>
          <td>semnum</td>
          <td>一般写0</td>
      </tr>
      <tr>
          <td>cmd</td>
          <td>SETVAL:初始化信号量<br> IPC_RMID：删除信号量</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>


### shell操作信号量
```C
ipcs   -s 查看消息队列
ipcrm  -s  msgid  删除消息队列
```

## 示例代码

```C
union semun
{
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

int sem_id;

//初始化信号量
int set_semvalue()
{
    union semun sem_union;    
    sem_union.val = 1;
    if(semctl(sem_id,0,SETVAL,sem_union)==-1)
        return 0;
    return 1;
}

//等待信号量
int semaphore_p()
{
    struct sembuf sem_b;
    sem_b.sem_num = 0;
    sem_b.sem_op = -1;
    sem_b.sem_flg = SEM_UNDO;
    if(semop(sem_id,&sem_b,1)==-1)
    {
        fprintf(stderr,"semaphore_p failed\n");
        return 0;
    }
    return 1;
}

//发送信号量
int semaphore_v()
{
    struct sembuf sem_b;
    sem_b.sem_num = 0;
    sem_b.sem_op = 1;
    sem_b.sem_flg = SEM_UNDO;
    if(semop(sem_id,&sem_b,1)==-1)
    {
        fprintf(stderr,"semaphore_v failed\n");
        return 0;
    }
    return 1;
}

//删除信号量
void del_semvalue()
{

    union semun sem_union;
    if(semctl(sem_id,0,IPC_RMID,sem_union)==-1)
        fprintf(stderr,"Failed to delete semaphore\n");
}

int main(int argc,char *argv[])
{
    char message = 'x';
    //创建信号量
     sem_id = semget((key_t)1234,1,0666|IPC_CREAT);
    if(argc>1)
    {
        //初始化信号量
        if(!set_semvalue())
        {
            fprintf(stderr,"init failed\n");
            exit(EXIT_FAILURE);
        }
        //参数的第一个字符赋给message
        message = argv[1][0];
    }
    int i=0;
    for(i=0;i<5;i++)
    {
        //等待信号量
        if(!semaphore_p())
            exit(EXIT_FAILURE);
        printf("%c",message);
        fflush(stdout);
        sleep(1);
        //发送信号量
        if(!semaphore_v())
            exit(EXIT_FAILURE);
        sleep(1);
    }
    printf("\n%d-finished\n",getpid());
    if(argc>1)
    {
        //退出前删除信号量
        del_semvalue();
    }
    exit(EXIT_SUCCESS);
}

```

# 共享内存

## 简介

共享内存就是允许两个或多个进程共享一定的存储区。就如同 malloc() 函数向不同进程返回了指向同一个物理内存区域的指针。当一个进程改变了这块地址中的内容的时候，其它进程都会察觉到这个更改。因为数据不需要在客户机和服务器端之间复制，数据直接写到内存，不用若干次数据拷贝，所以这是最快的一种IPC。

> 注：共享内存没有任何的同步与互斥机制，所以要使用信号量来实现对共享内存的存取的同步。

## 函数

### shmget
```C
int shmget(key_t key, size_t size, int shmflg);
功能：根据键值创建或者打开共享内存
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
          <td>key</td>
          <td>ftoke返回的键值</td>
          <td rowspan="4" >无</td>
      </tr>
      <tr>
          <td>size</td>
          <td>该共享存储段的长度(字节)</td>
      </tr>
      <tr>
          <td>shmflg</td>
          <td>标识函数的行为及共享内存的权限类似</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回共享内存的标识符，失败返回-1</td>
      </tr>
    </tbody>
</table>

### shmat
```C
void *shmat(int shmid, const void *shmaddr, int shmflg);
功能：共享内存的映射
一般用法：void addr = shmat(shmid,NULL,0);shmid为shmget返回的键值，后两个参数这样写表明系统自动分配
成功返回映射到当前进程的地址，失败返回-1
```


### shmdt
```C
int shmdt(const void *shmaddr);
功能：解除映射，只是和当前进程分离，不删除共享内存
shmaddr：shmat函数映射到当前进程的地址
成功返回0，失败返回-1
```

### shmctl
```C
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
功能：共享内存控制函数
对共享内存进行各种控制，如修改共享内存的属性，或删除共享内存。其中参数含义和消息队列类似常用的也就是删除共享内存,如：
shmctl(shmid,IPC_RMID,NULL);
成功返回0；错误返回-1
```

### shell中操作共享内存
```C
查看：ipcs   -m
删除：ipcrm -m  shmid
```

## 示例代码

```C
shmdata.h的源代码如下：
#ifndef _SHMDATA_H_HEADER
#define _SHMDATA_H_HEADER
#define TEXT_SZ 2048
struct shared_use_st
{  
    int written;//作为一个标志，非0：表示可读，0表示可写 
    char text[TEXT_SZ];//记录写入和读取的文本
};
#endif



源文件shmread.c的源代码如下：

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/shm.h>
#include "shmdata.h"
int main()
{  
    int running = 1;//程序是否继续运行的标志  
    void *shm = NULL;//分配的共享内存的原始首地址   
    struct shared_use_st *shared;//指向shm   
    int shmid;//共享内存标识符 //创建共享内存   
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
    if(shmid == -1)
    {      
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }   //将共享内存连接到当前进程的地址空间
    shm = shmat(shmid, 0, 0);
    if(shm == (void*)-1)   
    {  
        fprintf(stderr, "shmat failed\n"); 
        exit(EXIT_FAILURE);
    }  
    printf("\nMemory attached at %X\n", (int)shm);  //设置共享内存   
    shared = (struct shared_use_st*)shm;   
    shared->written = 0;
    while(running)//读取共享内存中的数据 
    {       //没有进程向共享内存定数据有数据可读取       
        if(shared->written != 0)
        {      
            printf("You wrote: %s", shared->text);      
            sleep(rand() % 3);          //读取完数据，设置written使共享内存段可写
            shared->written = 0;         //输入了end，退出循环（程序）  
            if(strncmp(shared->text, "end", 3) == 0)    
                running = 0;       
        }      
        else//有其他进程在写数据，不能读取数据     
            sleep(1);  
    }   //把共享内存从当前进程中分离
    if(shmdt(shm) == -1)   
    {      
        fprintf(stderr, "shmdt failed\n");     
        exit(EXIT_FAILURE);
    }   //删除共享内存   
    if(shmctl(shmid, IPC_RMID, 0) == -1)   
    {  
        fprintf(stderr, "shmctl(IPC_RMID) failed\n");  
        exit(EXIT_FAILURE);
    }  
    exit(EXIT_SUCCESS);
}

源文件shmwrite.c的源代码如下：

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>
#include "shmdata.h"
int main()
{  
    int running = 1;   
    void *shm = NULL;  
    struct shared_use_st *shared = NULL;
    char buffer[BUFSIZ + 1];//用于保存输入的文本
    int shmid;  //创建共享内存
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
    if(shmid == -1)
    {  
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }   //将共享内存连接到当前进程的地址空间
    shm = shmat(shmid, (void*)0, 0);   
    if(shm == (void*)-1)
    {  
        fprintf(stderr, "shmat failed\n");     
        exit(EXIT_FAILURE);
    }  
    printf("Memory attached at %X\n", (int)shm);    //设置共享内存   
    shared = (struct shared_use_st*)shm;   
    while(running)//向共享内存中写数据  
    {       //数据还没有被读取，则等待数据被读取,不能向共享内存中写入文本       
        while(shared->written == 1)     
        {          
            sleep(1);      
            printf("Waiting...\n");
        }       //向共享内存中写入数据       
        printf("Enter some text: ");       
        fgets(buffer, BUFSIZ, stdin);      
        strncpy(shared->text, buffer, TEXT_SZ);      //写完数据，设置written使共享内存段可读       
        shared->written = 1;     //输入了end，退出循环（程序）  
        if(strncmp(buffer, "end", 3) == 0)         
            running = 0;   
    }   //把共享内存从当前进程中分离
    if(shmdt(shm) == -1)   
    {      
        fprintf(stderr, "shmdt failed\n");     
        exit(EXIT_FAILURE);
    }  
    sleep(2);  
    exit(EXIT_SUCCESS);
}

```

---