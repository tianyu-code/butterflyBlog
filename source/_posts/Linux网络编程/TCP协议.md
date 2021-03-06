---
title: TCP协议
date: 2020-08-19 12:50:32
tags:
  - 网络编程
  - TCP协议
categories:
  - Linux网络编程

keywords: Linux,网络编程,TCP协议
cover: /images/封面图/TCPUDP.jfif
top_img: /images/封面图/TCPUDP.jfif
description: 本文介绍TCP协议内容，包括帧格式、可靠传输工作原理及实现、流量控制和拥塞控制。
sticky: 0

---

#  概念
**传输控制协议（TCP，Transmission Control Protocol）**是一种面向连接的、可靠的、基于字节流的传输层通信协议。主要特点如下：
1. TCP是面向连接的运输层协议。应用程序在使用TCP协议之前，必须先建立TCP连接。在传递数据完毕后，必须释放已建立的TCP连接。
2. 每一条TCP连接只能有两个端点，只能是点对点的。
3. TCP提供可靠交付的服务，通过TCP连接传送的数据，无差错，不丢失，不重复，并且按序到达。
4. TCP提供全双工通信。TCP允许通信双方的应用进程在任何时候都能发送数据。TCP连接的两端都设有发送缓存和接收缓存，用来临时存放双向通信的数据。
5. 面向字节流。TCP中的“流”指的是流入到进程或从进程流出的字节序列。“面向字节流”的含义是：虽然应用程序和TCP的交互是一次一个数据块(大小不等)，但TCP把应用程序交下来的数据看成仅仅是一连串的无结构的字节流。TCP并不知道所传送的字节流的含义。TCP不保证接收方应用程序所收到的数据块和发送方应用程序所发出的数据块具有对应大小的关系。但接收方应用程序收到的字节流必须和发送方应用程序发出的字节流完全一样。当然，接收方的应用程序必须有能力识别收到的字节流，把它还原成有意义的应用层数据。

面向字节流的理解如下图所示：
<img src=/images/TCP协议/字节流.png>

+ TCP和UDP在发送报文时所采用的方式完全不同。TCP并不关心应用程序一次把多长的报文发送到TCP缓存中，而是根据对方给出的窗口值和当前网络拥塞的程度来决定一个报文段应包含多少个字节（UDP发送的报文长度是应用程序给出的）。
+ 如果应用程序传送到TCP缓存的数据块太大，TCP就可以把它划分短一些再传。TCP也可以等待积累有足够多的字节后再构建成报文段发送出去。

# TCP帧结构


<img src="/images/报文格式/TCP.png" >

+ **源端口和目的端口：**各占2个字节，分别写入源端口号和目的端口号。
+ **序列号(Sequence　Number)：**占4字节，在TCP连接中传送的字节流中的每一个字节都按顺序编号，整个要传送的字节流的其实序号必须在连接建立时设置。该字段则指本报问段所发送的数据的第一个字节的序号。例如，一段报文的序列号字段值为301，携带数据100字节，那么下一个报文段的序列号字段应为401.
    > 通过序列号，TCP接收端可以识别出重复接收到的TCP包，从而丢弃重复包，同时对于乱序数据包也可以依靠系列号进行重排序，进而对高层提供有序的数据流。另外如果接收的包中包含SYN或FIN标志位，逻辑上也占用1个byte，应答号需加1。
+ **确认号(Acknowledgment   Number)：**占4字节，是期望收到对方下一个报文段的第一个数据字节的序号。例如，接收端接收到一个净荷为12byte的数据包，SN为5，则会回复一个确认收到的数据包，如果这个数据包之前的数据也都已经收到了，这个数据包中的确认号则设置为12+5=17，表示之前的数据都已经收到了，准备接受SN=17的数据包。
+ **头部长度：**占4位，指示TCP头的长度，即数据从何处开始。单位为4字节，即TCP头部长度最大值为60字节
+ **保留**：占6位，保留今后使用，目前应置为0.
+ **标志：**6位标志位，下面介绍：
    1. **紧急 URG(Urgent)：**当URG=1时，表明**紧急指针**字段生效。它告诉系统此报文中由紧急数据，应尽快传送，而不要按原来的排队顺序来传送。当URG=1时，发送应用进程就告诉发送方的TCP由紧急数据要传送。于是发送方TCP就把紧急数据插入到本报问段数据的**最前面**，而在紧急数据后面的数据仍是普通数据，这时要与首部中的**紧急指针**字段配合使用。
    2. **确认 ACK(ACKnowlegment)：**仅当ACK=1时，**确认号**字段才生效。TCP规定，在连接建立后所有传送的报文段都必须将ACK置1。
    3. **推送 PSH(Push)：**该标志置位时，一般是表示发送端缓存中已经没有待发送的数据，**接收端不将该数据进行队列处理**，而是尽可能快将数据转由应用处理。在处理 `telnet` 或 `rlogin` 等交互模式的连接时，该标志总是置位的,目的是希望键入一个命令后立即就能收到对方的响应。
    4. **复位 RST(Reset)：**当RST=1时，表明TCP连接中出现严重差错，必须释放连接，然后再重新建立连接。RST置1还用来拒绝一个非法的报文段或拒绝打开一个连接
    5. **同步 SYN(SYNchronization)：**再连接建立时用来同步序号。当SYN=1而ACK=0时，表明这是一个连接请求报文段。若对方同意建立连接，则应再响应的报文段中使SYN=1和ACK=1。
    6. **终止 FIN(Finish)：**用来释放一个连接。当FIN=1时，表明此报文段的发送方的数据已经发送完毕，并要求释放运输连接
+ **窗口尺寸：**占2字节，窗口尺寸是作为接收方让发送方设置其发送窗口的依据，用于TCP的流量控制。
+ **校验和：**占2字节，校验和字段检查的范围包括首部和数据两部分。和UDP一样，再计算校验和时，要在TCP报文的前面加上12字节的伪首部，其格式与UDP一样，只不过需要将第4个字段的17改为6（TCP的协议号是6），把第5字段中的UDP长度改为TCP长度，计算方式也和UDP一致。
+ **紧急指针：**占2字节，仅当URG=1时才有意义，它指出本报文段中的紧急数据的字节数（紧急数据结束后就是普通数据）。
    > 注意，即使窗口值为0也可发送紧急数据 
+ **选项：**长度不定，但长度必须以是32bits的整数倍。常见的选项包括MSS(最大报文段长度)、SACK（选择确认）、Timestamp（时间戳）等等。


# 可靠传输工作原理

我们知道，TCP发送的报文段是交给IP层传送的。但IP层只能提供尽最大努力服务，因此TCP应采用适当的措施使通信变得可靠。   
理想的传输条件有以下两个特点：
1. 传输信道不产生差错
2. 不管发送方以多快的速度发送数据，接收方总是能来得及处理收到的数据。
这样的理想传输条件下，不需要采用任何措施就能实现可靠传输。然后实际的网络不具备上述条件，所以需要一些适当的措施。

## 停止等待协议
### 简介
<img src=/images/TCP协议/停止等待.png>

上图中a为无差错情况，b为出现差错，注意：
+ 在发送完一个分组后，必须暂时保留已发送的分组的副本（为发生超时重传时使用）。只有收到响应的确认后才能清除分组副本。
+ 分组和确认分组都必须进行编号，这样才能确认哪个发送的分组收到的确认，哪个没有。
+ 超时计时器的重传时间应当比数据在分组传输的平均往返时间更长一些。

### 确认丢失和确认迟到

<img src=/images/TCP协议/确认丢失.png>

上图中a表示确认报文丢失的情况，b表示确认报文迟到的情况:
+ 当确认M1的数据包丢失时，A经过一段超时时间后重传M1，B接收并丢弃重复的M1之后，重传确认M1数据包；
+ 当B发送的确认M1数据包由于网络原因迟到了，在A端规定的超时时间内未到达A，A端就会重传M1，B接收并丢弃重复的M1之后，重传确认M1数据包，并继续通信。当迟到的确认M1数据包到达A时，A收下数据包但什么也不做

> 上述的确认和重传机制常成为自动重传请求ARQ(Automatic Repeat reQuest)，意思是重传的请求是自动进行的，接收方不需要请求发送方重传某个出错的分组

## 连续ARQ协议

由于停止等待协议的信道利用率较低，而采用流水线传输，如下图所示，当使用流水线传输时，就要使用下面介绍的连续ARQ协议和滑动窗口协议
<img src=/images/TCP协议/连续ARQ.png>

滑动窗口协议比较复杂，这里先介绍连续ARQ协议最基本的概念，详细在后续实现时介绍
<img src=/images/TCP协议/发送窗口.png>

1. a表示发送方的发送窗口，表示位于发送窗口内的5个分组都可以连续的发送出去，而不需要等待对方的确认，这样信道利用率就提高了
2. 连续ARQ协议规定，发送方每收到一个确认，就把发送窗口向前滑动一个分组的位置，如图b所示。
3. 接收方一般采用累积确认的方式，也就是说，接收方不必对收到的分组逐个发送确认，而是在收到几个分组后，对按序到达的最后一个分组发送确认，表示到这个分组为止的所有分组都正确收到了

> 累积确认有的优点是：容易实现，即使确认丢失也不必重传。缺点是：不能向发送方反映出接收方已经正确收到的有分组的信息

# 可靠传输的实现

本节为可靠传输的实现，即现实中应用的技术

## 以字节为单位的滑动窗口

<img src=/images/TCP协议/滑动窗口.png>

如上图所示，现假定场景为A收到了B发来的确认报文段，其中窗口是20，确认号是31（这表明B期望收到的下一个序号是31，而30为止的数据已经收到），该场景下A的发送窗口即为上图所示。   

**发送方A的发送窗口：**
+ 发送窗口表示：在没有收到B的确认的情况下，A可以连续把窗口内的数据都发送出去，凡是已经发送过的数据，在未收到确认之前都必须暂时保留 以便在超时重传时使用
+ 后沿的后面表示已发送且已收到确认，这些数据显然不需要再保留。前沿的前面部分表示不允许发送。
+ 发送窗口的位置由前沿和后沿共同确认。发送窗口的前沿通常是不断前移，也有可能不动：一是没有收到新的确认，且窗口大小不变；二是收到新的确认但是窗口变小，正好使得前沿不动
    > 前沿也有可能向后收缩，这发生在对方通知的窗口缩小了，但TCP标准强烈不赞成这样做，因为很可能发送方在收到这个通知以前已经发送了窗口中的许多数据，现在又要收缩窗口，不让发送这些数据，这样就会产生一些错误


<img src=/images/TCP协议/滑动窗口2.png>

现假定A发送了序号为31~41的数据，如图上图所示：

+ 从上可知，要描述一个发送窗口的状态需要三个指针：P1,P2和P3，意义如下
    + 小于P1的是已发送并收到确认的部分，而大于P3的是不允许发送的部分
    + P3-P1=A的发送窗口（又叫通知窗口）
    + P2-P1=已发送但尚未收到确认的字节数
    + P3-P2=允许发送但尚未发送的字节数（又叫可用窗口或有效窗口）
+ 再看B的接收窗口，B的接收窗口大小未20，接收窗口外面，到30号位置的数据是已经发送过确认，并且已经交付主机了，接收窗口内的序号（31-50）是允许接收的，在上图中，B收到了序号32和33的数据，这些数据没有按序到达，因为31未收到（也许丢失了，也许滞留在网络中），注意，B只能对按序收到的数据中的最高序号给出确认，因此B发送的确认报文段中的确认号仍未31

<img src=/images/TCP协议/滑动窗口3.png>

如上图所示，假定B收到了序号为31的数据，并把序号31-33的数据交付主机，然后B删除这些数据，接着将接收窗口向前移动3个序号，同时给A发送确认，其中窗口值仍未20，但确认号是34，这表明B已经收到了到序号33位置的数据   
A收到B的确认后，就可以把窗口向前滑动3个序号，但指针P2不动

<img src=/images/TCP协议/滑动窗口4.png>

如上图所示，A在继续发送完序号42-53的数据后，P2向前移动和P3重合，发送窗口内的序号都已用完但还没有收到确认，此时必须停止发送。
注意：可能B已经收到了数据，且给出了确认，但是这些确认滞留在网络中，此时A需要经过一段时间后重传这部分数据，直到收到B的确认为止。


**总结补充：**
+ A 的发送窗口并不总是和 B 的接收窗口一样大（因为有一定的时间滞后）。
+ TCP 标准没有规定对不按序到达的数据应如何处理。通常是先临时存放在接收窗口中，等到字节流中所缺少的字节收到后，再按序交付上层的应用进程。
+ TCP 要求接收方必须有累积确认的功能，这样可以减小传输开销


## 超时重传时间的选择
待补充

## 选择确认SACK

若收到的报文段无差错，只是未按序号，中间还缺少一些序号的数据，那么能否设法只传送缺少的数据而不重传已经正确到达接收方的数据？选择确认就是一种可行的方法，下面举例说明

<img src=/images/TCP协议/SACK.png>

接收方收到了如上图所示的字节块，如果这些字节的序号都在接收窗口之内，那么接收方先手下这些数据，但要把这些信息准确地告诉发送方，使发送方不要再重复发送这些已收到的数据

该方法需要在TCP首部的选项中加上“**允许SACK**”的选项，而双方必须都实现商定好，若使用选择确认，那么原来首部中的“**确认号字段**”的用法不变，在选项中报告收到的不连续的字节块边界，由于选项的长度最长未40字节，且指明一个连接需要4字节（因为序号是4字节的），那么在选项中最多指明4个字节块的边界信息（8个边界=32字节，一个字节指明SACK选项，一个字节指明该选项占用多少字节）

> 然而SACK文档未指明发送方应当如何响应SACK，因此大多数的实现还是重传所有未被确认的数据块。


# 流量控制

流量控制(flow control)就是让发送方的发送速率不要太快，既要让接收方来得及接收，也不要使网络发生拥塞

## 利用滑动窗口实现流量控制

下面举例说明，设A向B发送数据，在连接建立时，B告诉A接收方的接收窗口rwnd=400，因此发送方的发送窗口不能超过接收方给出的接收窗口的数值。注意，窗口的单位是字节不是报文段，数据发送过程如下图所示：

<img src=/images/TCP协议/流量控制.png>

明显的可以看到主机B进行了三次流量控制，即减小rwnd的值来进行流量控制。

注意：若B向A发送了零窗口的报文段后不久，B的接收缓存又有了一些空间，于是B向A发送rwnd=400的报文段，但是这个报文丢失了，导致A一直等待B发送非零窗口的通知，而B一直等待A发送数据，导致**死锁**。

为了解决这个问题，TCP为每个连接设有一个**持续计时器(persistence timer)**,只要TCP连接的一方收到对方的零窗口通知，就启动 持续计时器，若设置的时间到期，就发送一个**零窗口探测报文段**（仅携带1字节的数据），而对方会回复当前的窗口值。若还是零窗口，那么重置计时器。

## 考虑传输效率

TCP报文段的发送时机：
1. 当发送缓存到达最大报文段长度MSS时，就发送
2. 发送应用进程指明推送操作（将TCP首部中PUSH置位），立即发送报文
3. 发送方计时器期限到了，将已有的缓存数据装入报文段（但长度不能超过MSS）发送出去

但如何控制TCP报文段的发送时机仍然是一个较为复杂的问题：例如一个交互式用户使用一条TELNET连接，若交互的数据只有一个字符，那么此时的报文传输效率就很低（头部占比较大）

在TCP的实现中广泛使用**Nagle算法**：
+ 若发送进程把要发送的数据逐个字节的送到缓存中，则发送方就把第一个字节先发出去，后面的先缓存起来，当收到对第一个字节的确认后，再把发送缓存中的所有数据组装成一个报文段发送出去，同时继续对随后到达的数据进行缓存，只有收到对前一个报文段的确认后才继续发送下一个报文段
+ 该算法还规定，当缓存中的数据已达到发送窗口大小的一半或已达到报文段的最大长度时，就立即发送一个报文段，这样可以提高网络的吞吐量

# 拥塞控制

## 概念

1. 拥塞：在某段时间，若对网络中某资源的需求超过了该资源所能提供的可用部分，网络的性能就要变坏——产生拥塞(congestion)。
2. 拥塞控制：防止过多的数据注入到网络中，这样可以使网络中的路由器或链路不致过载。
3. 拥塞控制和流量控制的区别
    + 拥塞控制是一个全局性的过程，涉及到所有的主机、所有的路由器，以及与降低网络传输性能有关的所有因素
    + 流量控制往往指在给定的发送端和接收端之间的点对点通信量的控制，它所要做的就是抑制发送端发送数据的速率，以便使接收端来得及接收
    > 举例说明流量控制和拥塞控制的区别：若某个光纤网络传输速率1000Gb/s，有一个巨型计算机向一个PC以1Gb/s的速率传输文件，显然网络带宽是够的，不存在拥塞问题，但巨型计算机必须时常停下来以便使PC机来得及接收，该场景即为流量控制；   
    若另一个网络，其链路传输速率为1Mbs，而有1000台计算机连接在这个网络上，假定其中500台向其余500台以100kb/s的速率发送文件，则此时接收方是来得及接受的，但整个网络的输入负载超过了网络承受极限，该场景即为拥塞控制

4. 拥塞控制的效果
<img src=/images/TCP协议/拥塞控制效果.png>

    上图给出了拥塞控制所起到的效果，其中横坐标为提供的负载，代表单位时间内**输入**给网络的分组数目，纵坐标为吞吐量，代表单位时间内从网络**输出**的分组数目:
    + 理想的拥塞控制为：随着传输数据量（负载）的增加，数据传输速度（吞吐量）越来越快，当吞吐量达到网络的最大带宽时，继续增大传输数据量，传输的速度（吞吐量）也不会再增加，所以这样并不会造成网络拥塞导致传输速度（吞吐量）下降
    + 无拥塞控制时，随着提供的负载的增大，网络吞吐量的增长速度逐渐减小，也就是说在网络吞吐量未达到饱和时，就有一部分分组被丢弃了，当网络的吞吐量明显小于理想的吞吐量时，网络就进入了轻度拥塞的状态，当提供的负载达到某一数值时，网络的吞吐量反而随提供的负载的增大而下降，此时网络就进入了拥塞状态，若继续增大负载，则会进入死锁
    + 实际的拥塞控制：在实际的拥塞控制中，网络设备随着传输的数据量（提供的负载）越来越大，传输的速度（吞吐量）越快，丢包率也越来越大，依靠拥塞控制机制会适当降低数据传输的速度（吞吐量），以减少拥塞

## 拥塞控制方法

四种算法：慢开始、拥塞避免、快重传和快恢复   
为了方便理解，下面假设：数据单项传送，另一个方向只传送确认；接收方总有足够大的缓存空间，因而发送窗口的大小由网络的拥塞程序来决定

### 慢开始和拥塞避免

发送方维持一个叫做**拥塞窗口**cwnd (congestion window)的状态变量。   
拥塞窗口的大小取决于网络的拥塞程度，并且动态地在变化。发送方让自己的发送窗口等于拥塞窗口。如再考虑到接收方的接收能力，则发送窗口还可能小于拥塞窗口。

**发送方控制拥塞窗口的原则是**：

+ 只要网络没有出现拥塞，拥塞窗口就再增大一些，以便把更多的分组发送出去。
+ 只要网络出现拥塞，拥塞窗口就减小一些，以减少注入到网络中的分组数。


**慢开始**：由小到大逐渐增大发送窗口（拥塞窗口），通常在刚刚开始发送报文段时，将拥塞窗口cwnd设置为一个最大报文段MSS的数值，而在**每收到**一个对新的报文段的确认后，将cwnd增加至多一个MSS的数值。   
下面举例说明慢开始的原理，为了方便起见，我们用报文段的个数作为窗口大小的单位（实际为字节）

<img src=/images/TCP协议/慢开始.png>

+ 第一轮：刚开始cwnd=1，发送第一个报文段M1，接收方收到后确认M1，发送方收到对M1的确认后，将cwnd从1增大到2
+ 第二轮：发送M2和M3，接收方收到后发回对M2和M3的确认，发送方每收到一个对新报文段的确认（重传的不算）就使发送方的拥塞窗口加1，所以此时cwnd从2增大到4
+ 第三轮：发送M4-M7报文段
+ 。。。。

由此可见，使用慢开始算法，每经过一个**传输轮次（即往返时间RTT）**，拥塞窗口cwnd就**加倍**

为了防止拥塞窗口增长过大引起网络拥塞，还需要设置一个**慢开始门限ssthresh**，用法如下:
+ 当 cwnd < ssthresh 时，使用慢开始算法（cwnd每轮加倍）。
+ 当 cwnd > ssthresh 时，停止使用慢开始算法而改用拥塞避免算法。
+ 当 cwnd = ssthresh 时，既可使用慢开始算法，也可使用拥塞避免算法。
 
**拥塞避免算法**的思路是让拥塞窗口 cwnd **缓慢地增大**，即每经过一个往返时间 RTT 就把发送方的拥塞窗口cwnd**加1**，而不是加倍，使拥塞窗口cwnd按线性规律缓慢增长。

**当网络出现拥塞时：**
+ 无论处于慢开始阶段还是拥塞避免阶段，只要发送方判断网络出现拥塞（**其根据就是没有按时收到确认数据包**），就要把慢开始门限 ssthresh 设置为出现拥塞时的发送方发送窗口值（多数情况下等于cwnd）的**一半**（但不能小于2）。
+ 然后把拥塞窗口 cwnd **重新设置为 1**，执行慢开始算法。
+ 这样做的目的就是要迅速减少主机发送到网络中的分组数，使得发生拥塞的路由器有足够时间把队列中积压的分组处理完毕

<img src=/images/TCP协议/慢开始和拥塞控制算法.png>

+ 可以看到，开始时采用慢开始算法，每经历一轮cwnd翻倍，传输的数据包翻倍，到第4轮时cwnd=16，意味着可以同时发送16个数据包，到达了设定的慢开始门限值ssthresh，随后采用拥塞避免算法。
+ 采用拥塞避免算法期间，每经历一轮cwnd+1。当cwnd=24时，发送方不能准时收到确认数据包（即丢包），判断出现网络拥塞，于是把慢开始门限值ssthresh重新设为当前cwnd值（24）的一半，即12，并把cwnd重置为1 ，再次进入慢开始阶段。

> “拥塞避免”并非指完全能够避免了拥塞。利用以上的措施要完全避免网络拥塞还是不可能的。   
“拥塞避免”是说在拥塞避免阶段把拥塞窗口控制为按线性规律增长，使网络比较不容易出现拥塞


### 快重传

**快重传的目的**：为了使发送方**尽早**知道有报文段没有到达对方，而无需等到超时计时器时限到达再重传数据报。

举例说明：   
+ 如下图所示，接收方收到M1和M2后都分别发出了确认，现假定接收方没有收到M3，但接着收到了M4，显然接收方不能确认M4，因为M4是收到的失序报文段，根据可靠传输原理，接收方可以什么都不做。
+ 但按照快重传算法的规定，**接收方应及时发送**对M2的重复确认，这样做可以让发送方尽早知道M3没有到达接收方。发送方接着发送M5和M6，接收方收到后，再次发出对M2的重复确认，
+ 快重传算法规定，只要发送方**一连收到3个**重复确认，就应当**立即重传**对方尚未收到的报文段M3.

<img src=/images/TCP协议/快重传.png>

### 快恢复

当发送方收到**3个连续的重复确认**后，就把慢开始门限ssthresh减半，这是为了预防网络出现拥塞   
由于发送王现在认为网络很有可能没有发生拥塞（若发生了严重的拥塞，就不会一连有好几个报文段连续到达接收方，也就不会导致接收方连续发送重复确认），因此之后不执行慢开始算法（即拥塞窗口cwnd现在不设置为1），而是把cwnd设置为慢开始门限ssthresh减半后的数值，然后执行拥塞避免算法，如下图所示

<img src=/images/TCP协议/快恢复.png>


# TCP连接的管理

## 连接建立-三次握手

<img src=/images/TCP协议/三次握手.png>

**过程介绍：**
1. 初始状态：客户端已经创建好套接字，并且知道服务器的端口和IP；服务器已经socket、bind、listen、accept了
2. 客户端调用connect函数发出连接请求（阻塞），之后发送一个序列号为x的报文，将SYN置1，注意，这个报文不能携带数据，但要消耗一个序列号，此时客户端进入**SYN-SENT（同步已发送）**状态
3. 服务器收到了客户端的请求，如果同意建立连接，就发送一个序列号为y的应答报文，确认号是x+1，将SYN和ACK位置1，这个报文也不能传送数据，但消耗一个序列号，此时服务器进入**SYN-RCVD（同步收到）**状态
4. 客户端收到确认后，还需要给服务器一个确认，将ACK置1，序列号为x+1，确认号为y+1，TCP规定ACK报文可以携带数据，如果未携带数据，则不消耗序列号，即下一个报文序列号还是x+1，这时TCP连接已经建立，A进入**ESTABLISHED（已建立连接）**状态。
5. B收到A的确认后，也进入**ESTABLISHED**状态


## 为什么采用三次握手呢？   
+ 正常情况下：A发出的请求丢失了，发生超时重传，然后B应答正确建立连接，
+ 异常情况：假如上述情况中的请求报文没有丢失，而是在网络中滞留了，以至于A对B的数据交换已经结束了，释放连接后才到B，B发出一个ACK，而A并不理会这个确认报文，假如没有三次握手，那么这时B认为连接建立，占用资源，而A却已经忽略这次连接，导致资源白白浪费。

## 连接释放-四次挥手

<img src=/images/TCP协议/四次挥手.png>

> 注：数据传送结束后，双方都可以结束连接

**过程介绍：**
1. A发送释放报文给B，并停止发送数据，主动关闭TCP连接，将FIN置1，序列号为u，u等于前面的序列号+1，注意，FIN报文即使不携带数据也消耗一个序列号，此时A进入**FIN-WAIT-1（终止等待1）**状态，等待B的确认
2. B返回一个应答，将ACK置1，序列号为w，确认号为u+1，此时B进入**CLOSE-WAIT（关闭等待）**状态。这个时候A到B的连接已经释放，但是B还可以给A正常发送数据
3. A收到确认报文后进入**FIN-WAIT-2（终止等待2）**状态，等待B发送释放报文
4. 等B没有数据要发送时，就发送释放报文，将FIN和ACK置1，序列号为w，确认号还是u+1（这里和上一个的确认报文的内容是一样的，因为A已经不能发送数据了），此时B进入**LAST-ACK（最后确认）**状态，等待A的确认
5. A收到B的连接释放报文后，发出确认报文，将ACK置1，序列号为u+1，确认号为w+1，此时进入**TIME-WAIT（时间等待）**状态，请注意，现在连接还没有释放掉，必须经过时间等待计时器设置的2个MSL(**最长报文段寿命**)后，A才关闭连接


## 为什么要等待2MSL的时间呢？

1. 为了保证A发送的最后一个ACK能够到达B。如果B没有收到确认报文会超时重传FIN报文，A接收到后会重置定时器，继续等待，看有没有B的超时重传
2. 为了防止“已失效的连接请求报文”。经过2MSL后，可使本次连接中的报文段都从网络中消失，避免对下次连接的影响。

> 这样可以发现，B结束TCP连接的时间比A早一些

> 注意：四次挥手的中间两次报文有可能是一次报文，比如客户端创建连接后立即释放连接，这时服务器没有什么要发的，就把中间两次的数据包放在一起了


# 有限状态机

<img src=/images/TCP协议/TCP状态机.png>

---