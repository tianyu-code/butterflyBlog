---
title: Linux基本套接字编程
date: 2020-08-21 11:00:32
tags:
  - 网络编程
  - Linux基本套接字编程
categories:
  - Linux网络编程

keywords: Linux,网络编程,Linux基本套接字编程
cover: /images/封面图/coding.jpg
top_img: /images/封面图/coding.jpg
description: 本文介绍Linux下编程的基本套接字函数。
sticky: 0

---



# socket
```C
int socket(int domain, int type, int protocol);
功能：创建套接字
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
          <td>domain</td>
          <td>设置网络通信的域，函数根据这个参数选择通信协议的族。常用参数AF_INET、AF_INET分别代表IPV4和IPV6</td>
          <td rowspan="4" >补充：type参数自从Linux2.6.27开始，还可指定socket的行为，分别为SOCK_NONBLOCK，SOCK_CLOEXEC</td>
      </tr>
      <tr>
          <td>type</td>
          <td>用于设置套接字通信的类型，常用参数有：<br>
            SOCK_STREAM：流式套接字，提供顺序的，可靠的，双向的，基于连接的字节流，常用TCP<br>
            SOCK_DGRAM ：数据报套接字，支持数据报（最大长度固定的无连接，不可靠消息），常用UDP<br>
            SOCK_RAW：原始套接字，提供原始网络协议访问</td>
      </tr>
      <tr>
          <td>protocol</td>
          <td>用于指定某个协议，常写0，自动选择，更多用法待补充</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回套接字的文件描述符，失败返回-1</td>
      </tr>
    </tbody>
</table>

# setsockopt/getsockopt

```C
int setsockopt(int socket, int level, int option_name,
              const void *option_value, socklen_t option_len);
int getsockopt(int sockfd, int level, int optname,
                      void *optval, socklen_t *optlen);
功能：设置/获取套接字属性，这两个函数的参数意义相同，见下表
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
          <td align="center" valign="middle" rowspan="5">参数</td>
          <td>socket</td>
          <td>待修改属性的套接字</td>
          <td rowspan="6" >该函数参数说明见下表,<br>
          表中get/set代表该选项是否支持getsockopt/setsockopt<br>
          套接字选项粗分为两大类：<br>
          一是启用或禁止某个特性的二元选项（下表中“标志”给出二元选项），<br>
          二是获取并返回我们可以设置或检查的特定值的选项<br>
          备注：本文中仅列出了作者目前可能遇到的选项，待后续学习补充</td>
      </tr>
      <tr>
          <td>level</td>
          <td>指定系统中解释后面选项参数的影响范围：<br>
          SOL_SOCKET: 通用套接字选项<br>
          IPPROTO_IP: IPv4套接字选项<br>
          IPPROTO_TCP: TCP套接字选项<br>
          其余暂不列出，待补充
          </td>
      </tr>
      <tr>
          <td>option_name</td>
          <td>选项名，具体参数见下表</td>
      </tr>
      <tr>
          <td>option_value</td>
          <td>具体选项对应的输入/输出数据，该参数为指针</td>
      </tr>
      <tr>
          <td>option_len</td>
          <td>option_value的大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>

---

## level：SOL_SOCKET
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;">option_name</th>
          <th align="center" valign="middle" style="width: 20px;">get</th>
          <th align="center" valign="middle" style="width: 20px;">set</th>
          <th align="center" valign="middle">说明</th>
          <th align="center" valign="middle" style="width: 60px;">标志</th>
          <th align="center" valign="middle" style="width: 100px;">数据类型</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>SO_BROADCAST</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>允许发送广播数据报，只有使用UDP协议并且开启此选项，才能发送广播报文，该标志目的是为了防止用户错误的输入目的地址为广播地址，此时若未开启此选项则sendto返回EACCES错误</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>
      <tr>
          <td>SO_DEBUG</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>开启跟踪调试，该选项仅由TCP支持</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>SO_DONTROUTE</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>绕过外出路由表查询，即数据报只发往直接相连的主机，不经路由器转发，不可达时sendto会返回ENETUNREACH错误</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>SO_ERROR</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle"></td>
          <td>获取待处理错误并清除，当一个套接字上发生错误时，源自Berkeley的内核中的协议模块将该套接字的名为so_error的变量设为标准的Unix Exxx值中的一个，我们称它为该套接字的待处理错误（pending error），内核能够以下面两种方式之一立即通知进程这个错误：<br>
①如果进程阻塞在对该套接字的select调用上，那么无论是检查可读条件还是可写条件，select均返回并设置其中一个或所有两个条件<br>
②如果进程使用信号驱动式I/O模型，那就给进程或进程组产生一个SIGIO信号。 进程然后可以通过访问SO_ERROR套接字选项获取so_error的值。由getsockopt返回的整数值就是该套接字的待处理错误。so_error随后由内核复位为0</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>SO_KEEPALIVE</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>周期性的测试连接是否仍存活，用于TCP，若两小时连接双方均无数据，则TCP自动给对端发送一个保持存活探测分解（keep-alive probe）</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>SO_LINGER</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>若有数据待发送则延迟关闭</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">linger{}</td>
      </tr>  
      <tr>
          <td>SO_OOBINLINE</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>让接收到的带外数据将被留在正常的输入队列中（即在线留存）</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>SO_RCVBUF</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>接收缓冲区大小</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr> 
      <tr>
          <td>SO_SNDBUF</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>发送缓冲区大小</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr>
      <tr>
          <td>SO_RCVLOWAT</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>接收低水位标记</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr> 
      <tr>
          <td>SO_SNDLOWAT</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>发送低水位标记</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr> 
      <tr>
          <td>SO_RCVTIMEO</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>接收超时</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">timeval{}</td>
      </tr>   
      <tr>
          <td>SO_SNDTIMEO</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>发送超时</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">timeval{}</td>
      </tr>
      <tr>
          <td>SO_REUSEADDR</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>允许重用本地地址</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>
      <tr>
          <td>SO_REUSEPORT </td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>允许重用本地端口</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>SO_TYPE </td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle"></td>
          <td>返回套接字的类型，例如SOCK_STREAM或SOCK_DGRAM</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr>
      <tr>
          <td>SO _USELOOPBACK </td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>路由套接字取得所发送数据的副本</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>       
    </tbody>
</table>



## level：IPPROTO_IP
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;">option_name</th>
          <th align="center" valign="middle" style="width: 20px;">get</th>
          <th align="center" valign="middle" style="width: 20px;">set</th>
          <th align="center" valign="middle">说明</th>
          <th align="center" valign="middle" style="width: 60px;">标志</th>
          <th align="center" valign="middle" style="width: 100px;">数据类型</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>IP_HDRINCL</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>随数据包含的IP首部</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr> 
      <tr>
          <td>IP_OPTIONS</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>IP首部选项</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">待补充</td>
      </tr>   
      <tr>
          <td>IP_RECVDSTADDR</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>返回目的IP地址</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>IP_RECVIF</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>返回接收接口索引</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>IP_TOS</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>服务类型和优先权</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr>  
      <tr>
          <td>IP_TTL</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>存活时间</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr> 
      <tr>
          <td>IP_MUTICAST_IF</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>指定外出接口</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">in_addr{}</td>
      </tr>
      <tr>
          <td>IP_MULTICAST_TTL</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>指定外出TTL</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">u_char</td>
      </tr> 
      <tr>
          <td>IP_MULTICAST_LOOP</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>发送低水位标记</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">u_char</td>
      </tr> 
      <tr>
          <td>IP_ADD_MEMBERSHIP</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">√</td>
          <td>加入多播组</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">ip_mreq{}</td>
      </tr>       
      <tr>
          <td>IP_DROP_MEMBERSHIP</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">√</td>
          <td>离开多播组</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">ip_mreq{}</td>
      </tr> 
      <tr>
          <td>IP_BLOCK_SOURCE</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">√</td>
          <td>阻塞多播组</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">ip_mreq_source{}</td>
      </tr> 
      <tr>
          <td>IP_UNBLOCKSOURCE</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">√</td>
          <td>解除阻塞多播组</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">ip_mreq_source{}</td>
      </tr> 
      <tr>
          <td>IP_ADD_SOURCE_MEMBERSHIP</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">√</td>
          <td>加入源特定多播组</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">ip_mreq_source{}</td>
      </tr> 
      <tr>
          <td>IP_DROP_SOURCE_MEMBERSHIP</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">√</td>
          <td>离开源特定多播组</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">ip_mreq_source{}</td>
      </tr> 
    </tbody>
</table>


## level：IPPROTO_IP
<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;">option_name</th>
          <th align="center" valign="middle" style="width: 20px;">get</th>
          <th align="center" valign="middle" style="width: 20px;">set</th>
          <th align="center" valign="middle">说明</th>
          <th align="center" valign="middle" style="width: 60px;">标志</th>
          <th align="center" valign="middle" style="width: 100px;">数据类型</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td>TCP_MAXSEG</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>TCP最大分节大小</td>
          <td align="center" valign="middle"></td>
          <td align="center" valign="middle">int</td>
      </tr> 
      <tr>
          <td>TCP_NODELAY</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">√</td>
          <td>禁止Nagle算法</td>
          <td align="center" valign="middle">√</td>
          <td align="center" valign="middle">int</td>
      </tr> 
    </tbody>
</table>

# bind
```C
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
功能：将本地协议地址与sockfd绑定
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>socket</td>
          <td>待绑定的套接字</td>
          <td rowspan="4" >套接字绑定的地址，地址的内容可以只指定一个端口号，或只指定一个IP地址，或者IP和端口同时指定，IP地址必须属于其所在主机的网络接口之一。<br><br>
            1.若不调用该函数绑定IP地址和端口<br>
            （1）服务器在启动时捆绑它们的众所周知端口。如果一个TCP客户或服务器未曾调用bind捆绑一个端口，当调用connect或listen时，内核就要为相应的套接字选择一个临时端口。让内核来选择临时端口对于TCP客户来说是正常的，除非应用需要一个预留端口；然而对于TCP服务器来说却极为罕见，因为服务器是通过它们的众所周知端口被大家认识的<br>
            （2）对于TCP客户端通常不绑定IP地址，当连接套接字时，内核将根据所用外出网络接口来选择源IP地址，而所用外出接口取决于到达服务器所需路径，若TCP服务器未绑定IP地址，内核就把客户端发送的SYN的目的IP地址作为服务器的源IP地址<br>
            2.若调用该函数时，端口号为0，则内核为其分配一个临时端口<br>
            3.若调用该函数时，IP地址为统配地址（INADDR_ANY,即0），那么内核将等到套接字已连接（TCP）或已在套接字上发出数据报（UDP）时才选择一个本地IP地址</td>
      </tr>
      <tr>
          <td>addr</td>
          <td>指向于一个特定协议的结构体指针</td>
      </tr>
      <tr>
          <td>addrlen</td>
          <td>addr的大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1，错误值在errno中指出，通常是EADDRINUSE（“Address already in use”），此时需要setsockopt的SO_REUSEADDR选项
</td>
      </tr>
    </tbody>
</table>

# connect

## 函数说明
```C
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
功能：客户端通过调用connect函数来建立与服务器的连接
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>socket</td>
          <td>待连接的套接字</td>
          <td rowspan="4" >该函数常用于TCP协议，但UDP套接字其实也可使用该函数，具体说明见下文</td>
      </tr>
      <tr>
          <td>addr</td>
          <td>要建立连接的服务器地址</td>
      </tr>
      <tr>
          <td>addrlen</td>
          <td>addr的大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>

## 注意事项

1. 如果是TCP套接字，调用`connect`函数将激发TCP的**三路握手过程**，而且仅在连接建立成功或出错时才返回，其中**出错返回**可能有以下几种情况：
    + 若TCP客户**没有收到**SYN分节的**响应**，则返回**ETIMEDOUT**错误。

    + 若对客户的SYN的响应是**RST**（表示复位），则表明该服务器主机在我们指定的端口 没有进程在等待与之连接（例如服务器进程也许没在运行）。这是一种**硬错误（hard error）**，客户一接收到RST就马上返回**ECONNREFUSED**错误
        > RST是TCP在发生错误时发送的一种TCP分节。产生RST的三个条件是：目的地为某端口的 SYN到达，然而该端口上没有正在监听的服务器（如前所述）；TCP想取消一个已有连接；TCP 接收到一个根本不存在的连接上的分节。

    + 若客户发出的SYN在中间的某个路由器上引发了一个“destination unreachable”（目的地不可达）ICMP错误，则认为是一种**软错误（soft error）**。客户主机内核保存该消息，并按第一种情况中所述的时间间隔继续发送SYN。若在某个规定的时间（4.4BSD规定75s）后仍未收到响应，则把保存的消息（即ICMP错误）作为**EHOSTUNREACH或ENETUNREACH**错误返回给进程。以下两种情形也是有可能的：
        + 一是按照本地系统的转发表，根本没有到达远程系统的路径
        + 二是 connect调用根本不等待就返回

2. 按照TCP状态转换图，connect函数导致当前套接字从CLOSED状态（该套接字自从由socket函数创建以来一直所处的状态）转移到SYN_SENT状态，若成功则再转移到ESTABLISHED状态。若connect失败则该套接字不再可用，必须关闭，我们不能对这样的套接字再次调用connect函数。在代码设计时，当循环调用函数connect为给定主机尝试各个IP地址直到有一个成功时，在每次connect失败后，都必须**close当前的套接字描述符并重新调用socket**

3. 如果是UDP套接字，则不会触发三次握手过程，一旦UDP套接字调用了connect系统调用，那么这个UDP上的连接就变成**一对一的连接**，但是通过这个UDP连接传输数据的性质还是不变的，**仍然是不可靠的UDP连接**。一旦变成一对一的连接，在调用系统调用发送和接受数据时也就可以使用TCP那一套系统调用了。

    + 我们**再也不能给输出操作指定目的IP地址和端口号**。也就是说，我们不使用sendto，而改用write或send。写到已连接UDP套接字上的任何内容都自动发送到由connect指定的协议地址。可以给已连接的UDP套接字调用sendto，但是不能指定目的地址。sendto的第五个参数必须为空指针，第六个参数应该为0.

    + 不必使用recvfrom以获悉数据报的发送者，而改用read、recv或recvmsg。在一个已连接UDP套接字上，由内核为输入操作返回的数据报只有那些来自connect指定协议地址的数据报。这样就限制一个已连接UDP套接字能且仅能与一个对端交换数据报。
    + 由已连接UDP套接字引发的异步错误会返回给它们所在的进程，而未连接的UDP套接字不接收任何异步错误。
    + 当有一个已连接UDP套接字的进程可出于下列两个目的之一再次调用connect:
        + 指定新的IP地址和端口号。
        + 为了断开一个已UDP套接字连接，再次调用connect时把套接字地址结构的地址族设置为AF_UNSPEC。

> UDP客户进程或服务器进程只在使用自己的UDP套接字与确定的唯一对端进行通信时，才可以调用connect。


# listen

```C
int listen(int sockfd, int backlog);
功能：将套接字变主动为被动
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
          <td>sockfd</td>
          <td>待操作套接字</td>
          <td rowspan="3" >该函数仅由TCP服务器调用，应在socket和bind函数后，accept函数前调用.<br>
          </td>
      </tr>
      <tr>
          <td>backlog</td>
          <td>规定内核应该为响应套接字排队的最大连接个数</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>

为了理解其中的backlog参数，我们必须认识到内核为任何一个给定的监听套接字维护两个队列：
+ **未完成连接队列（incomplete connection queue）**，每个这样的SYN分节对应其中一项： 已由某个客户发出并到达服务器，而服务器正在等待完成相应的TCP三路握手过程。这些套接字处于SYN_RCVD状态
+ **已完成连接队列（completed connection queue）**，每个已完成TCP三路握手过程的客户对 应其中一项。这些套接字处于ESTABLISHED状态

下图描绘了监听套接字的这两个队列：
<img src=/images/Linux基本套接字编程/等待队列.png>

> 从Linux2.2内核开始，backlog参数代表已完成连接队列。未完成连接队列大小由/proc/sys/net/ipv4/tcp_max_syn_backlog文件修改，若backlog参数比/proc/sys/net/core/somaxconn中的值大，那么内核会自动将其校准为/proc/sys/net/core/somaxconn中的值


# accept
```C
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
功能：从已连接队列中取出一个已经建立的连接，如果没有任何连接可用，则进入睡眠等待(阻塞)
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
          <td>sockfd</td>
          <td>待操作套接字</td>
          <td rowspan="4" >1.该函数由TCP服务器调用<br>
            2.若对客户端的地址不感兴趣，则地址参数可全填写为NULL<br>
            3.后续的数据收发操作皆基于该函数返回的已连接套接字<br>
            4.如果没有连接请求在等待，accept会阻塞直到一个请求到来。如果sockfd处于非阻塞模式，accept会返回-1，并将errno设置为EAGAIN或EWOULDBLOCK</td>
      </tr>
      <tr>
          <td>addr</td>
          <td>用来保存已连接的对端程序的地址</td>
      </tr>
      <tr>
          <td>addrlen</td>
          <td>addr参数的大小，函数返回时，该参数指出实际返回的地址大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>

# send/sendto

## 函数说明

```C
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                      const struct sockaddr *dest_addr, socklen_t addrlen);
功能：发送数据
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" >说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="6">参数</td>
          <td>sockfd</td>
          <td>待操作套接字</td>
          <td rowspan="7" ></td>
      </tr>
      <tr>
          <td>buf</td>
          <td>待发送数据</td>
      </tr>
      <tr>
          <td>len</td>
          <td>发送数据长度</td>
      </tr>
      <tr>
          <td>flags</td>
          <td>MSG_CONFIRM：指示数据链路层协议支持监听对方的回应，直到得到答复，它仅能用于SOCK_DGRAM和SOCK_RAW类型的socket（待日后补充理解）<br><br>
            MSG_DONTROUTE：本标志告知内核目的主机在某个直接连接的本地网络上，因而无需执行路由表查找，直接将数据发送到本地局域网内的主机。和setsockopt的SO_DONTROUTE设置一致，区别是MSG_DONTROUTE为单词操作<br><br>
            MSG_DONTWAIT：本标志在无需打开相应套接字的非阻塞标志的前提下，把单个I/O操作临时指定为非阻塞，接着执行I/O操作，然后关闭非阻塞标志。<br><br>
            MSG_MORE：告诉内核应用程序还有更多数据要发送，内核将超时等待新数据写入TCP发送缓冲区后一并发送。这样可以放置TCP发送过多的报文段，从而提高传输效率<br><br>
            MSG_NOSIGNAL：往读端关闭的管道或者socket连接中写数据时不引发SIGPIPE信号<br><br>
            MSG_OOB：对于send，本标志指明即将发送带外数据。</td>
      </tr>
      <tr>
          <td>dest_addr</td>
          <td>目的地址</td>
      </tr>
      <tr>
          <td>addrlen</td>
          <td>dest_addr参数大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回发送的字节数，失败返回-1</td>
      </tr>
    </tbody>
</table>

## 注意事项
1. UDP支持发送长度为0的数据，TCP不支持（长度为0代表关闭连接）
2. send函数只有在connect后的套接字才可使用（这样才知道预期的接收者是谁，因为send函数中没有目的地址）
send和write的唯一区别就是flags的存在，send和sendto的唯一区别就是目的地址的存在
3. 当套接字为已connect模式的，那么使用sendto函数的dest_addr和addrlen必须为NULL和0，否则返回错误EISCONN，反之亦然，若dest_addr和addrlen是NULL和0时，发现套接字不是已连接状态的，也会返回ENOTCONN错误
4. 如果消息太长而无法通过底层协议，则返回错误EMSGSIZE，并且不发送消息
5. 当消息不适合套接字的发送缓冲区时，除非套接字已置于非阻塞I / O模式，否则send（）通常会阻塞。 在非阻塞模式下，在这种情况下，它将失败并显示错误EAGAIN或EWOULDBLOCK。 select调用可用于确定何时可以发送更多数据

# recv/recvfrom
```C
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                        struct sockaddr *src_addr, socklen_t *addrlen);
功能：接收数据
```

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 75px;"> </th>
          <th align="center" valign="middle" style="width: 75px;">名称</th>
          <th align="center" valign="middle" >说明</th>
          <th align="center" valign="middle">备注</th>
      </tr>
    </thead>
    <tbody>
      <tr>
          <td align="center" valign="middle" rowspan="6">参数</td>
          <td>sockfd</td>
          <td>待操作套接字</td>
          <td rowspan="7" ></td>
      </tr>
      <tr>
          <td>buf</td>
          <td>保存接收到的数据</td>
      </tr>
      <tr>
          <td>len</td>
          <td>欲读取的数据长度，实际读取的字节数可能少于该参数</td>
      </tr>
      <tr>
          <td>flags</td>
          <td>MSG_DONTWAIT：本次操作为非阻塞<br>
MSG_OOB：对于recv，本标志指明即将接收带外数据。<br>
MSG_PEEK：本标志适用于recv和recvfrom，它允许我们查看缓冲区中的数据，而且系统不在recv或recvfrom返回后丢弃这些数据。<br>
MSG_WAITALL：它告知内核不要在尚未读入请求数目的字节之前让一个读操作返回<br>
即使指定了MSG_WAITALL，如果发生下列情况之一：(a)捕获一个信号， (b)连接被终止，(c)套接字发生一个错误，相应的读函数仍有可能返回比 所请求字节数要少的数据</td>
      </tr>
      <tr>
          <td>src_addr</td>
          <td>数据来源地址，若不关心可填NULL</td>
      </tr>
      <tr>
          <td>addrlen</td>
          <td>src_addr参数大小，若不关心可填NULL</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回接收的字节数，失败返回-1<br>
            若套接字连接对端关闭，则该函数成功返回0<br>
            若UDP发送长度为0的数据时，该函数也返回0</td>
      </tr>
    </tbody>
</table>

# 获取地址函数
```C
int getsockname(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
功能：获取套接字的本地地址

int getpeername(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
功能：获取连接对方的协议地址

上述两个函数除了功能不同，参数意义均相同，
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
          <td align="center" valign="middle" rowspan="3">参数</td>
          <td>sockfd</td>
          <td>待操作套接字</td>
          <td rowspan="4" >注意：如果提供的缓冲区太小，返回的地址将被截断。 在这种情况下，addrlen将返回一个大于输入的值<br>
          getsockname应用场景：<br>
            1.在一个没有调用bind的TCP客户上，connect成功后使用该函数返回由内核赋予该连接的本地IP地址和本地端口号<br>
            2.在以端口号0调用bind（告知内核去选择本地端口号）后，getsockname用于返回由内 核赋予的本地端口号<br>
            3.在一个以通配IP地址调用bind的TCP服务器上，与某个客户的连接一旦建立 （accept成功返回），getsockname就可以用于返回由内核赋予该连接的本地IP地址。在这样的调用中，套接字描述符参数必须是已连接套接字的描述符，而不是监听套接字的描述符<br><br>
        getpeername应用场景：<br>
        当一个服务器是由调用过accept的某个进程通过调用exec执行程序时，它能够获取客户身份的唯一途径便是调用getpeername</td>
      </tr>
      <tr>
          <td>addr</td>
          <td>获取的套接字地址结果由该参数返回</td>
      </tr>
      <tr>
          <td>addrlen</td>
          <td>指出addr参数的大小，函数返回时，该参数指出实际返回的地址大小</td>
      </tr>
      <tr>
          <td align="center" valign="middle">返回值</td>
          <td>int</td>
          <td>成功返回0，失败返回-1</td>
      </tr>
    </tbody>
</table>












---