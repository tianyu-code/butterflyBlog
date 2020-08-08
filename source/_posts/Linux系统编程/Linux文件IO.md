---
title: Linux文件IO
date: 2020-08-07 14:36:32
tags:
  - Linux
  - 文件IO
categories:
  - Linux系统编程

keywords: Linux,系统编程,文件IO,文件描述符,缓冲
cover: /images/封面图/coding.jpg
top_img: /images/封面图/coding.jpg
description: 本文介绍文件IO的相关函数使用以及文件描述符和缓冲的概念介绍。
sticky: 0

---


#  文件描述符

Linux系统将**所有设备都当作文件来处理**，而Linux用**文件描述符**来标识每个文件对象。

文件描述符在形式上是一个非负整数。实际上，它是一个**索引值**，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。

## 标准文件描述符

每个进程都有一张文件描述符的表，进程刚被创建时，**标准输入、标准输出、标准错误输出**设备文件被打开，对应的文件描述符0、1、2 记录在表中。在进程中打开其他文件时，系统会返回文件描述符表中最小可用的文件描述符，并将此文件描述符记录在表中

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" bgcolor=#00a8f3>文件描述符</th>
          <th align="center" valign="middle" bgcolor=#00a8f3>缩写</th>
          <th align="center" valign="middle" bgcolor=#00a8f3>描述</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>0</td>
          <td>STDIN</td>
          <td>标准输入</td>
      </tr>
      <tr>
          <td>1</td>
          <td>STDOUT</td>
          <td>标准输出</td>
      </tr>
      <tr>
          <td>2</td>
          <td>STDERR</td>
          <td>标准错误</td>
      </tr>
    </tbody>
</table>


## 文件描述符的复制

### dup

```C
int dup(int oldfd);
功能：复制旧的文件描述符，自动分配一个可用的最小的文件描述符
成功返回新的文件描述符，失败返回-1
```
**函数说明：**
它们引用相同的打开文件描述，因此共享文件偏移量和文件状态标志。 例如，如果在旧的文件描述符之上使用`lseek`修改了文件偏移，则新的也将更改。

> 关于文件描述符与打开文件、文件的关系在后续文章将会介绍，阅读后能更容易理解上述说明

### dup2
```C
int dup2(int oldfd, int newfd);
功能：复制旧的文件描述符，自动分配一个可用的最小的文件描述符
成功返回新的文件描述符，失败返回-1
```

**函数说明：**
1. `dup2`是`dup`函数的升级版本，可以指定生成的文件描述符（必须小于1024）,如果这个指定的描述符已经打开了，那么会原子地关闭和复制。
2. 若`oldfd`是无效的，则`newfd`不会被关闭
3. 若`oldfd`是无效的，且`newfd`和`oldfd`相等，则`dup2`函数什么也不干，直接返回`newfd`


### 修改标准文件描述符

标准文件描述符0，1，2一旦被改变了就无法使用了，所以在重定向之前需要把他们三个保存起来：
```C
int new_stdout = dup(1);//在重定向之前保存起来
dup2(new_stdout,1);//这样就可以变回来了
```

写例子的时候还有个小问题，我们重定向之后，`printf`到文件当中，然后把`stdout`变回来，再`printf`一句话，这个时候可以看到终端上两句话都打印出来了，那是因为重定向输出到文件的时候缓冲区是全缓冲的，所以数据还在缓冲区当中，没有写到文件当中呢，为了避免这类问题，可以选择使用系统调用（无缓冲区）

> 关于缓冲区的问题可继续阅读本文后续章节

## exec后的文件描述符

无论是`fork`还是`system`出子进程，如果父进程里在`open`某个文件后（包括`socket fd`）没有设置`FD_CLOEXEC`标志，就会引起各种不可预料的问题,特别是`socket的fd`本身又包括了本机ip，端口号等信息资源，如果该`socket fd`被子进程继承并占用，或者未关闭，就会导致新的父进程重新启动时不能正常使用这些网络端口，严重的就是设备掉线。

打开文件后默认未将该标志位置位，即默认在exec后不关闭文件描述符，可进行如下设置：
```C
int flags;
flags = fcntl(fd, F_GETFD);//获得标志
flags |= FD_CLOEXEC; //打开标志位
flags &= ~FD_CLOEXEC; //关闭标志位
fcntl(fd, F_SETFD, flags);//设置标志
```

> 其实open函数的flag提供了`O_CLOEXEC`标志位，可直接设置（仅Linux 2.6.23后支持）。

## 文件描述符和打开文件的关系

1. 每个文件描述符都指向一个打开的文件相对应
2. 不同的文件描述符可能指向同一个打开的文件
3. 相同的文件可能被不同的进程打开，也可以在被同一个进程打开多次


具体情况要具体分析，需要查看由内核维护的3个数据结构：
1. **进程级的文件描述符表：**进程级的列表，也就是用户区的一部分，进程每打开一个文件就会新建一个文件描述符，同时只能通过文件描述符的函数访问
2. **系统级的打开文件表：**系统级的列表，对当前系统的所有进程都共享.
3. **文件系统的i-node表：**inode索引节点表。


<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 30px;"> </th>
          <th align="center" valign="middle" style="width: 240px;">文件描述符表</th>
          <th align="center" valign="middle" style="width: 350px;">打开文件表</th>
          <th align="center" valign="middle">i-node表</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="9">记录内容</td>
          <td>文件描述符操作标志（目前内核仅定义了一个close-on-exec标志）</td>
          <td>当前文件偏移量</td>
          <td>文件类型</td>
      </tr>
      <tr>
          <td>对打开文件句柄的引用</td>
          <td>打开文件时使用的状态标识（open的flags参数）</td>
          <td>文件锁</td>          
      </tr>
      <tr>
          <td rowspan="7"></td>
          <td>文件访问模式（open时设置的O_RDONLY等标志）</td> 
          <td>文件拥有者的UID,GID</td>
      </tr>
      <tr>
          <td>对该文件i-node对象的引用</td> 
          <td>文件的时间戳:ctime,mtime,atime</td>
      </tr>
      <tr>
          <td>文件类型（例如：常规文件、套接字或FIFO）</td> 
          <td>链接数，即有多少文件名指向这个inode</td>
      </tr>
      <tr>
          <td>访问权限</td> 
          <td>读写执行权限</td>
      </tr>
      <tr>
          <td>一个指针，指向该文件所持有的锁列表</td> 
          <td>文件数据block的位置</td>
      </tr>
      <tr>
          <td>文件的各种属性，包括大小以及各种时间戳</td> 
          <td rowspan="2" >文件的各种属性，包括大小以及各种时间戳</td>           
      </tr>
      <tr>
          <td>与信号驱动相关的设置</td> 
      </tr>
    </tbody>
</table>

示例如下图所示：

<img src=/images/Linux文件IO/文件描述符和打开文件和inode的关系.png>

+ 在进程A中，文件描述符1和30都指向了同一个打开的文件句柄（标号23）。这可能是通过调用`dup、dup2`
+ 进程A的文件描述符2和进程B的文件描述符2都指向了同一个打开的文件句柄（标号73）。这种情形可能是在调用`fork`后出现的
+ 进程A的描述符0和进程B的描述符3分别指向不同的打开文件句柄，但这些句柄均指向i-node表的相同条目（1976），发生这种情况是因为每个进程各自对同一个文件发起了`open`调用，同一个进程两次打开同一个文件，也会发生类似情况

## 文件描述符限制

### 系统级限制

查看方式：
```bash
sysctl -a | grep -i file-max --color
cat /proc/sys/fs/file-max
```
`sysctl`命令和`proc`文件系统中查看到的数值是一样的，这属于系统级限制，它是限制所有用户打开文件描述符的总和

### 用户级限制

每个进程的最大文件描述符限制：
```bash
ulimit -n
```

### 修改方式

1. 修改用户级限制：

    ```bash
    ulimit -SHn 10240
    ```
    以上的修改只对当前会话起作用，是临时性的，如果需要永久修改，则要修改`/etc/security/limits.conf`文件：

    ```bash
    * soft nofile 100001
    * hard nofile 100002
    ```
    soft 指的是当前系统生效的设置值，hard 表明系统中所能设定的最大值

2. 修改系统级限制：

    ```shell
    [root@VM-0-4-centos ~]# cat /proc/sys/fs/file-max
    350000
    [root@VM-0-4-centos ~]# echo 50000 > /proc/sys/fs/file-max 
    [root@VM-0-4-centos ~]# cat /proc/sys/fs/file-max
    50000
    [root@VM-0-4-centos ~]# sysctl -a | grep -i file-max --color
    fs.file-max = 50000

    ```
    以上是临时修改,重启后失效，永久修改如下

    把`fs.file-max=400000`添加到`/etc/sysctl.conf`中，使用`sysctl -p`即可


# 缓冲区

出于速度和效率考虑，系统IO调用和标准 C语言库的IO函数均会对数据进行缓冲，接下来将分类介绍：

## 系统IO调用缓冲

`read`和`write`在操作磁盘文件的时候不会直接发起磁盘访问，而是在用户空间缓冲区和内核缓冲区高速缓存之间复制数据。
```C
write(fd,"abc",3);
```
上面的语句将3个字节的数据从用户空间内存传递到内核空间的缓冲区中，随后`write`返回，在后续的某个时刻，内核会将其缓冲区中的数据写入（刷新至）磁盘，在此期间如果有另一进程访问这几个字节，直接从高速缓存中提供这些数据。对输入而言同理。
这一设计不需要`read`和`write`等待磁盘操作，也减少了内核进行磁盘传输的次数。例如：让磁盘写1000次，每次写入一个字节，还是一次写入1000个字节，内核访问磁盘的次数都是相同的，因为有缓冲区的存在，但是我们更趋向于后者，因为只有一次系统调用，所以这部分是程序员需要思考的，这部分的缓冲也就是下面提到的stdio库的缓冲了。

> 简单来说，就是在write系统调用和实际的磁盘之间还有一层由内核维护的缓冲。

## stdio库的缓冲

在操作磁盘文件的时候，虽然有内核维护的缓冲来减少访问磁盘的次数以节省开销，但是还有一部分开销是由系统调用产生的，也就是程序中确定每次`write`或者`read`多少个字节，而stdio库的缓冲就是帮程序员干这件事的,分为以下三类：
1. 无缓冲
    每个stdio库函数立即调用`write`或者`read`

2. 行缓冲
    只带终端设备的流默认为这一缓冲类型。对于输出流，在输出一个换行符（除非缓冲区已经填满）前将缓冲数据，遇到换行符会刷新缓冲区。对于输入流，每次读取一行数据

3. 全缓冲
    单次读写数据（通过write和read）的大小和缓冲区相同，只带磁盘的流默认采用此模式。


## 手动刷新stdio缓冲区


```C
int fflush(FILE *stream);
```

1. 使用该库函数强制将stdio输出流中的数据刷新到内核缓冲区中
2. 应用于输入流时，这将丢弃已缓冲的输入数据。当程序下一次尝试从流中读取数据时，将重新装载缓冲区
3. 若`stream`为`NULL`，则将刷新所有的输出缓冲区
4. 当关闭流时，自动刷新缓冲区

# 函数

## open
```C
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
功能：打开pathname所标识的文件，并返回文件描述符，flags可以指定文件的打开方式，mode指定了访问权限，如果flags中没有创建文件的标志，mode可以忽略
```

flag取值：
<table>
    <thead>
      <tr>
          <th align="center" valign="middle">标志</th>
          <th align="center" valign="middle">用途</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>O_RDONLY</td>
          <td>以只读方式打开</td>
      </tr>
      <tr>
          <td>O_WRONLY</td>
          <td>以只写方式打开</td>
      </tr>
      <tr>
          <td>O_RDWR</td>
          <td>以读写方式打开</td>
      </tr>
      <tr>
          <td></td>
          <td></td>
      </tr>
      <tr>
          <td>O_CLOEXEC</td>
          <td>设置close-on-exec标志，默认关闭，即exec后文件描述符不关闭</td>
      </tr>
      <tr>
          <td>O_CREAT</td>
          <td>若文件不存在则创建，需要指定mode</td>
      </tr> 
      <tr>
          <td>O_EXCL</td>
          <td>结合O_CREAT标志使用，专门用于创建文件（若文件已存在，则直接返回错误）</td>
      </tr>
      <tr>
          <td>O_NONBLOCK</td>
          <td>以非阻塞方式打开</td>
      </tr>
      <tr>
          <td>O_APPEND</td>
          <td>总在文件尾追加数据（若多个进程同时对同一文件追加数据，可能导致文件损坏）</td>
      </tr>
      <tr>
          <td>O_TRUNC</td>
          <td>截断已有文件，使其长度为0</td>
      </tr>
      <tr>
          <td>O_SYNC</td>
          <td>以同步方式写入文件</td>
      </tr>
      <tr>
          <td>O_ASYNC</td>
          <td>当IO操作可行时，产生信号通知进程（此特性仅适用终端、伪终端、socket和管道）</td>
      </tr>
      <tr>
          <td>O_DSYNC</td>
          <td>提供同步的IO数据完整性，即write返回后，数据均已输出到硬件</td>
      </tr>
      <tr>
          <td>O_DIRECT</td>
          <td>无缓冲的输入输出</td>
      </tr>
      <tr>
          <td>O_DIRECTORY</td>
          <td>如果pathname不是目录，则失败</td>
      </tr>
      <tr>
          <td>O_LARGEFILE</td>
          <td>在32位系统中使用该标志打开大文件</td>
      </tr>
      <tr>
          <td>O_NOATIME</td>
          <td>调用read时不修改文件最近访问时间</td>
      </tr>
      <tr>
          <td>O_NOCTTY</td>
          <td>不要让pathname(所指向的终端设备)成为控制终端</td>
      </tr>
      <tr>
          <td>O_NOFOLLOW</td>
          <td>对符号链接不予解引用</td>
      </tr>
    </tbody>
</table>


补充：
1. O_DIRECT和O_SYNC的区别

    O_DIRECT：绕过内核的页面缓存将数据写入设备，但是设备本身也存在缓存所以并不能保证数据就一定固化到磁盘上
    O_SYNC：文件数据和所有文件元数据同步写入磁盘

2. O_DSYNC、O_RSYNC、O_SYNC的区别

    + Linux中无O_RSYNC，glibc定义O_RSYNC具有与O_SYNC相同的值
    + 在写操作中，O_DSYNC和O_SYNC均保证数据同步更新到文件中，O_DSYNC将仅保证刷新对文件长度元数据的更新（而O_SYNC也刷新最后的修改时间戳记元数据）

## read
```C
ssize_t read(int fd, void *buf, size_t count);
功能：从fd文件中读取至多count字节的数据并保存到buf中。
返回值为实际读取到的字节数，如再无字节可读（例如读到文件结尾符EOF时），返回值为0
```


## write
```C
ssize_t write(int fd, const void *buf, size_t count);
功能：从buf中读取多达count字节的数据写入fd指代的已打开的文件中
返回值为实际写入文件中的字节数，有可能小于count
```


## close
```C
int close(int fd);
功能：释放文件描述符fd及相关的内核资源
成功返回0，失败返回-1
```

## lseek
```C
off_t lseek(int fd, off_t offset, int whence);
功能：改变文件偏移量
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" style="width: 400px;">说明</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>fd</td>
          <td>文件描述符</td>
      </tr>
      <tr>
          <td>offset</td>
          <td>指定了一个以字节为单位的数值</td>
      </tr>
      <tr>
          <td>whence</td>
          <td>表示参照哪个基点来解释offset，取值如下：<br>
                SEEK_SET   文件开头<br>
                SEEK_CUR  当前偏移量<br>
                SEEK_END  文件末尾</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>off_t</td>
          <td>成功返回距文件开头的偏移量，失败返回-1</td>
      </tr>
    </tbody>
</table>

> lseek不适用于所有类型的文件，不能用于如管道、FIFO、socket和终端


# 文件空洞

如果程序的文件偏移量已经跨越了文件结尾，然后在执行I/O操作，将会发生`read`调用返回0，表示文件结尾，`write`可以正常写入数据。从文件结尾到重新用`write`写入数据的这段空间被称为**文件空洞**，从编程角度看，**文件空洞**中是存在字节的，读取空洞将会返回以0(空字节)填充的缓冲区。然而**文件空洞不会占用磁盘空间**。

+ `ls`命令可以查看文件在文件系统中的大小（逻辑大小），这个大小是**包含文件空洞**的空字节大小的.
+ `du`命令可以查看文件在磁盘中实际占用的空间，`du -s test`结果表示的是多少个1024字节




---