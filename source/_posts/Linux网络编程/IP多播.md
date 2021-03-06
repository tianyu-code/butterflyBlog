---
title: IP多播
date: 2020-08-18 21:50:32
tags:
  - 网络编程
  - IP多播
categories:
  - Linux网络编程

keywords: Linux,网络编程,IP多播
cover: /images/封面图/donate.jpg
top_img: /images/封面图/donate.jpg
description: 本文介绍IP多播的概念及编程方法。
sticky: 0

---

# 基本概念
与单播相比，在一对多的通信中，多播可大大节省网络资源，例如下图，左边使用单播方式向90个主机传送相同的视频节目，为此需要发送90个单播，而右边使用多播方式向属于同一个多播组的90个成员传送节目，这时只需将视频分组当作多播数据报来发送，且只需发送一次

<img src=/images/IP多播/多播区别.png>

在因特网范围内的多播要靠路由器来实现，这些路由器必须增加一些能够识别多播数据报的软件，这样的路由器叫做多播路由器。

# IP地址

我们知道在因特网中，每一个主机必须有一个全球唯一的IP地址，如果这个主机现在想要接收某个特定多播组的分组，要如何做？
1. 显然这个多播数据报的目的地址一定不能写入这个主机的IP地址，一个多播组中有成千上万的主机，不可能在数据报首部加入这么多的主机的IP地址。在多播数据报的目的地址写入的是**多播组的标识**，然后设法让加入到这个多播组的**主机的IP地址**与**多播组的标识符**关联起来
2. 这个标识符就是**D类地址**（前四位1110，范围224.0.0.0到239.255.255.255），因此，多播数据报和一般的IP数据报的区别就是它使用D类IP地址作为目的地址，且首部中的协议字段是2，表示使用网际组管理协议IGMP。
3. 多播地址只能用于目的地址，不能用于源地址。此外，对多播数据报不产生ICMP差错报文，因此若ping命令后面键入多播地址，将永远不会收到响应

> 可使用的多播地址：224.0.1.0-238.255.255.255


# 硬件多播

IP多播分为两种，一种是只在本局域网进行硬件多播，另一种则是在因特网的范围进行多播，第一种虽然简单但很重要，因为现在大部分主机都是通过局域网接入到因特网的，在因特网进行多播的最后阶段还是要把多播数据报在局域网上用硬件多播交付多播组的所有成员

D类地址和MAC地址的映射：

<img src=/images/IP多播/多播MAC映射.png>

只有23位IP地址映射成为了MAC地址，所以多播IP地址与以太网的硬件地址的映射关系不是唯一的，所以接收到多播数据报的主机，还要再IP层利用软件进行过滤

# IGMP

在互联网上向多播组上的主机传送多播数据报，需要利用IGMP协议，IGMP使多播路由器知道多播组成员信息。示例如下

<img src=/images/IP多播/IGMP.jpg>

1. IGMP协议并不是在互联网范围内对所有多播组成员进行管理的协议。IGMP不知道IP多播组包含的成员数，也不知道这些成员都分布在哪些网络上。IGMP协议是让连接在本地局域网上的多播路由器知道本地局域网是否有主机（严格讲，是主机的某个进程）参加或退出了某个多播组。
2. 显然，仅有IGMP协议是不能完成多播任务的。连接在局域网上的多播路由器还必须和互联网上的其他多播路由器协调工作，以便把多播数据报以最小代价传送给所有的多播组成员。这就需要使用**多播路由选择协议**，后节介绍。

IGMP的工作分为两个阶段：

+ 第1阶段：当某台主机加入新的多播组时，该主机应向多播组的多播地址发送一个IGMP报文，声明自己要成为该组的成员。本地的多播路由器收到IGMP报文后，还要利用多播路由选择协议把这种组成员关系转发给互联网上的其他多播路由器。
+ 第2阶段：由于多播组成员是动态的。本地多播路由器要周期性探询本地局域网上的主机，以便知道这些主机是否还继续是组成员。只要有一台主机对某个组响应了，那么多播路由器就认为该组仍是活跃的。但当一个组在经过多次的探询后仍然没有一台主机响应，多播路由器就认为本网络上的主机都已经离开这个多播组了，因此也就不再把这个组的成员关系转发给其他的多播路由器

# 多播路由协议

多播路由协议比单播要复杂的多，此处暂时不过多讲解，由一个例子简单理解：

<img src=/images/IP多播/多播路由协议.png>

假定上图由两个多播组，多播组（1）成员有主机ABC，多播组（2）有DEF，分布在N1，N2和N3上。

1. 路由器R不应当向N3网络转发多播组（1）的分组，因为N3上没有该组成员。但每个主机可随时加入或离开一个多播组，比如G现在加入了多播组（1），那么路由器R就需要向N3转发多播组（1）的分组，也就是说多播转发必须动态地适应多播组成员的变化（这时网络拓扑未发生变化）

2. 再比如，主机 EF都是多播组（2）的成员，当E向F发送多播数据时，路由器R把这个数据报转发到N3，但F向E发送多播数据报时，路由器R则把多播数据报转发到N2，若路由器R收到主机A的多播数据报（A不是多播组（２）的成员，但也可以向改组发送多播数据报），则路由器R应当把多播数据报转发到N２和N３，由此可见，多播路由器在转发多播数据报时，不能仅仅根据多播数据报中的目的地址，还要考虑这个多播数据报从什么地方来和要到什么地方去

3. 还有，主机G未参加任何多播组，但G却可向任何多播组 发送多播数据报。主机G所在的局域网上也可以没有任何多播组的成员，综上所述，多播数据报可以由未加入多播组的主机发出，也可以通过没有组成员接入的网络

# 总结

至此我们知道多播：

+ 对于发送端，需要以某个**D类地址为目的IP地址**，代表向**某个多播组**发送报文，而多播数据报经过多播路由器时，多播路由器还知道哪个网络中存在多播组成员（**该信息通过IGMP获得**）从而进行转发，到达目的局域网后若支持硬件多播，多播组成员即可接收到多播数据报

+ 对于接收端，多播组成员知道自己属于哪个多播组（加入时通过IGMP告知多播路由器），接收到多播数据报后，先在数据链路层进行基**于MAC地址的不完备过滤**：因为IP向MAC地址的映射不完全，导致了不同Ip映射出的MAC地址是相同的，所以网卡接收的时候不能完全确定是不是自己的数据包；然后在IP层比较该多播数据报的目的IP地址是不是自己加入的多播组。过程如下图所示“

<img src=/images/IP多播/多播过程.png width="70%" height="100%">

# Linux编程实现多播

在Linux中想要接收多播数据，可使用`setsockopt`函数来加入或离开多播组，如下所示:
```C
struct in_addr
{
    in_addr_t s_addr;
}

struct ip_mreq
{
    struct in_addr imr_multiaddr;//多播组IP
    struct in_addr imr_interface;//将要添加到多播组的IP
}
int setsockopt(int sockfd, int level,int optname, const void *optval, socklen_t optlen);
```
<img src=/images/IP多播/多播函数.png>



---