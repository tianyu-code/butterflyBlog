---
title: GDB调试之定位段错误
date: 2020-04-05 21:35:29
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, 定位段错误
description: 本文介绍定位程序的段错误的几种方法

---

#  **程序添加打印日志**

走查代码，逐步添加printf打印，逐步定位，该方法最简单，且在开发调试过程中也较为快捷有效

#  **GDB调试程序**

编译时加入-g参数，使用gdb调试运行程序，出现错误时直接打印堆栈信息来定位
```
[root@VM_0_4_centos example_breakInSo]# gdb main
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-115.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/yu.tian/stackBreak/example_breakInSo/main...done.
(gdb) r
Starting program: /home/yu.tian/stackBreak/example_breakInSo/main 
sum:3

Program received signal SIGSEGV, Segmentation fault.
0x00007ffff7bd9812 in add (para1=1, para2=2) at add.cpp:21
21	    memcpy((char *)0x9527, name, sizeof(name));
Missing separate debuginfos, use: debuginfo-install glibc-2.17-292.el7.x86_64 libgcc-4.8.5-39.el7.x86_64 libstdc++-4.8.5-39.el7.x86_64
(gdb) bt
#0  0x00007ffff7bd9812 in add (para1=1, para2=2) at add.cpp:21
#1  0x0000000000400634 in main () at main.c:10
(gdb) 
```
适用场景：开发调试过程中，方便重新编译和运行

#  **core文件**

开启core dump，`ulimi -c unlimited`,程序崩溃时会生成core文件，使用gdb查看堆栈信息定位

适用场景：
+ 可在开发调试中使用
+ 也可用于现场无源代码时快速定位错误位置
+ 无需重新运行程序


#  **内核打印日志**

其实段错误的core dump也是借助了内核的反异常代码生成了core文件，那么发生段错误的时候在`/var/log/messages`中也有相关的提示信息,可使用`dmesg` 查看内核打印
一般的形式为
`进程名[pid]: segfault at 错误地址 ip 错误指令 sp 错误堆栈 error 错误原因 in 错误发生的可执行文件名称[加载基地址+不知道是啥东西]`

例如
`main[18281]: segfault at 9527 ip 00007fdd5de94812 sp 00007fff486ea9d0 error 6 in libadd.so[7fdd5de94000+1000]`

错误原因由3个bit组成，含义如下：
+ **bit2**:值为1表示是用户态程序内存访问越界，值为0表示是内核态程序内存访问越界
+ **bit1**: 值为1表示是写操作导致内存访问越界，值为0表示是读操作导致内存访问越界
+ **bit0**: 值为1表示没有足够的权限访问非法地址的内容，值为0表示访问的非法地址根本没有对应的页面，也就是无效地址

我们可通过错误指令地址-动态库加载的基地址得到函数的相对地址,再配合`nm`找到地址附近的函数，使用`gdb`或者`objdump`反汇编后定位到函数内的具体位置

适用场景：
+ 不需要-g参数编译，不需要借助于core文件，不需要重新运行程序，但需要有一定的汇编语言基础
+ core文件中堆栈信息全为？？？？？？？？，时可尝试使用此方法来定位
+ 如果使用了gcc编译优化参数（-O1，-O2，-O3）的话，生成的汇编指令将会被优化，使得定位过程有些难度

实例如下：

肥肠简单的源代码，为了增大定位难度，简单的加了些无用代码
```
#include "add.h"
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
char name[] = "aaa12312312312312313123123123121231312123sdasfasasfafasdasdadasdasdadsad";

int testUp(int para)
{
    return 0;
}

int add(int para1, int para2)
{
    int a = 1,b = 2;
    int sum = a + b;
    printf("sum:%d\n",sum);
    a = 3;
    b = 4;
    usleep(1);
    memcpy((char *)0x9527, name, sizeof(name));

    usleep(10);
    sum = a + b;
    a = 0;
    b = 0;
    return a+b;
}

int testDown(int para)
{
    return 1;
}

```

执行程序,
```
[root@VM_0_4_centos example_breakInSo]# ./main
sum:3
Segmentation fault

```

出现了段错误，此时没有打开core文件，可查看内核打印信息

```
[root@VM_0_4_centos example_breakInSo]# dmesg | tail -1
[19739.211945] main[18281]: segfault at 9527 ip 00007fdd5de94812 sp 00007fff486ea9d0 error 6 in libadd.so[7fdd5de94000+1000]
```

可见发生错误的指令地址为`0x7fdd5de94812`,并且错误位置为`libadd.so`,该动态库加载的基地址为`0x7fdd5de94000`
所以错误函数的相对地址为`0x7fdd5de94812-0x7fdd5de94000 = 0x812`

接下来使用`nm`命令查看符号表，大概定位出现问题的函数

```
[root@VM_0_4_centos example_breakInSo]# nm libadd.so -n -l -C
                 w __cxa_finalize@@GLIBC_2.2.5
                 w __gmon_start__
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 w _Jv_RegisterClasses
                 U printf@@GLIBC_2.2.5
                 U usleep@@GLIBC_2.2.5
0000000000000648 T _init
00000000000006c0 t deregister_tm_clones
00000000000006f0 t register_tm_clones
0000000000000730 t __do_global_dtors_aux
0000000000000770 t frame_dummy
00000000000007a5 T testUp(int)	/home/yu.tian/stackBreak/example_breakInSo/add.cpp:8
00000000000007b3 T add(int, int)	/home/yu.tian/stackBreak/example_breakInSo/add.cpp:13
00000000000008b2 T testDown(int)	/home/yu.tian/stackBreak/example_breakInSo/add.cpp:31
00000000000008c0 T _fini
00000000000008d4 r __GNU_EH_FRAME_HDR
00000000000009a0 r __FRAME_END__
0000000000200dc0 t __frame_dummy_init_array_entry
0000000000200dc8 t __do_global_dtors_aux_fini_array_entry
0000000000200dd0 d __JCR_END__
0000000000200dd0 d __JCR_LIST__
0000000000200dd8 d __dso_handle
0000000000200de0 d _DYNAMIC
0000000000201000 d _GLOBAL_OFFSET_TABLE_
0000000000201040 D name	/home/yu.tian/stackBreak/example_breakInSo/add.cpp:6
0000000000201089 B __bss_start
0000000000201089 b completed.6355
0000000000201089 D _edata
0000000000201090 B _end
0000000000201090 d __TMC_END__

```
可以发现大致就是在add这个函数中出现错误嗒，那么接下来使用`objdump`命令反汇编来精确定位错误位置

```
[root@VM_0_4_centos example_breakInSo]# objdump libadd.so -S -C --start-address=0x7b3 --stop-address=0x8b2

libadd.so:     file format elf64-x86-64


Disassembly of section .text:

00000000000007b3 <add(int, int)>:
{
    return 0;
}

int add(int para1, int para2)
{
 7b3:	55                   	push   %rbp
 7b4:	48 89 e5             	mov    %rsp,%rbp
 7b7:	48 83 ec 20          	sub    $0x20,%rsp
 7bb:	89 7d ec             	mov    %edi,-0x14(%rbp)
 7be:	89 75 e8             	mov    %esi,-0x18(%rbp)
    int a = 1,b = 2;
 7c1:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
 7c8:	c7 45 f8 02 00 00 00 	movl   $0x2,-0x8(%rbp)
    int sum = a + b;
 7cf:	8b 45 f8             	mov    -0x8(%rbp),%eax
 7d2:	8b 55 fc             	mov    -0x4(%rbp),%edx
 7d5:	01 d0                	add    %edx,%eax
 7d7:	89 45 f4             	mov    %eax,-0xc(%rbp)
    printf("sum:%d\n",sum);
 7da:	8b 45 f4             	mov    -0xc(%rbp),%eax
 7dd:	89 c6                	mov    %eax,%esi
 7df:	48 8d 3d e3 00 00 00 	lea    0xe3(%rip),%rdi        # 8c9 <_fini+0x9>
 7e6:	b8 00 00 00 00       	mov    $0x0,%eax
 7eb:	e8 90 fe ff ff       	callq  680 <printf@plt>
    a = 3;
 7f0:	c7 45 fc 03 00 00 00 	movl   $0x3,-0x4(%rbp)
    b = 4;
 7f7:	c7 45 f8 04 00 00 00 	movl   $0x4,-0x8(%rbp)
    usleep(1);
 7fe:	bf 01 00 00 00       	mov    $0x1,%edi
 803:	e8 a8 fe ff ff       	callq  6b0 <usleep@plt>
    memcpy((char *)0x9527, name, sizeof(name));
 808:	48 8b 05 e9 07 20 00 	mov    0x2007e9(%rip),%rax        # 200ff8 <name@@Base-0x48>
 80f:	48 8b 10             	mov    (%rax),%rdx
 812:	48 89 14 25 27 95 00 	mov    %rdx,0x9527
 819:	00 
 81a:	48 8b 50 08          	mov    0x8(%rax),%rdx
 81e:	48 89 14 25 2f 95 00 	mov    %rdx,0x952f
 825:	00 
 826:	48 8b 50 10          	mov    0x10(%rax),%rdx
 82a:	48 89 14 25 37 95 00 	mov    %rdx,0x9537
 831:	00 
 832:	48 8b 50 18          	mov    0x18(%rax),%rdx
 836:	48 89 14 25 3f 95 00 	mov    %rdx,0x953f
 83d:	00 
 83e:	48 8b 50 20          	mov    0x20(%rax),%rdx
 842:	48 89 14 25 47 95 00 	mov    %rdx,0x9547
 849:	00 
 84a:	48 8b 50 28          	mov    0x28(%rax),%rdx
 84e:	48 89 14 25 4f 95 00 	mov    %rdx,0x954f
 855:	00 
 856:	48 8b 50 30          	mov    0x30(%rax),%rdx
 85a:	48 89 14 25 57 95 00 	mov    %rdx,0x9557
 861:	00 
 862:	48 8b 50 38          	mov    0x38(%rax),%rdx
 866:	48 89 14 25 5f 95 00 	mov    %rdx,0x955f
 86d:	00 
 86e:	48 8b 50 40          	mov    0x40(%rax),%rdx
 872:	48 89 14 25 67 95 00 	mov    %rdx,0x9567
 879:	00 
 87a:	0f b6 40 48          	movzbl 0x48(%rax),%eax
 87e:	88 04 25 6f 95 00 00 	mov    %al,0x956f

    usleep(10);
 885:	bf 0a 00 00 00       	mov    $0xa,%edi
 88a:	e8 21 fe ff ff       	callq  6b0 <usleep@plt>
    sum = a + b;
 88f:	8b 45 f8             	mov    -0x8(%rbp),%eax
 892:	8b 55 fc             	mov    -0x4(%rbp),%edx
 895:	01 d0                	add    %edx,%eax
 897:	89 45 f4             	mov    %eax,-0xc(%rbp)
    a = 0;
 89a:	c7 45 fc 00 00 00 00 	movl   $0x0,-0x4(%rbp)
    b = 0;
 8a1:	c7 45 f8 00 00 00 00 	movl   $0x0,-0x8(%rbp)
    return a+b;
 8a8:	8b 45 f8             	mov    -0x8(%rbp),%eax
 8ab:	8b 55 fc             	mov    -0x4(%rbp),%edx
 8ae:	01 d0                	add    %edx,%eax
}
 8b0:	c9                   	leaveq 
 8b1:	c3                   	retq   
[root@VM_0_4_centos example_breakInSo]#
```
可以找到错误地址`0x812`的具体位置啦，就是在这个memcpy处，可以看出确实是在向`0x9527`地址写内容的时候报错啦，和我们构建的错误一致

使用GDB的方法如下
```
[root@VM_0_4_centos example_breakInSo]# gdb libadd.so 
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-115.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/yu.tian/stackBreak/example_breakInSo/libadd.so...done.
(gdb) disas/m add
Dump of assembler code for function add(int, int):
14	{
   0x00000000000007b3 <+0>:	push   %rbp
   0x00000000000007b4 <+1>:	mov    %rsp,%rbp
   0x00000000000007b7 <+4>:	sub    $0x20,%rsp
   0x00000000000007bb <+8>:	mov    %edi,-0x14(%rbp)
   0x00000000000007be <+11>:	mov    %esi,-0x18(%rbp)

15	    int a = 1,b = 2;
   0x00000000000007c1 <+14>:	movl   $0x1,-0x4(%rbp)
   0x00000000000007c8 <+21>:	movl   $0x2,-0x8(%rbp)

16	    int sum = a + b;
   0x00000000000007cf <+28>:	mov    -0x8(%rbp),%eax
   0x00000000000007d2 <+31>:	mov    -0x4(%rbp),%edx
   0x00000000000007d5 <+34>:	add    %edx,%eax
   0x00000000000007d7 <+36>:	mov    %eax,-0xc(%rbp)

17	    printf("sum:%d\n",sum);
   0x00000000000007da <+39>:	mov    -0xc(%rbp),%eax
   0x00000000000007dd <+42>:	mov    %eax,%esi
   0x00000000000007df <+44>:	lea    0xe3(%rip),%rdi        # 0x8c9
   0x00000000000007e6 <+51>:	mov    $0x0,%eax
   0x00000000000007eb <+56>:	callq  0x680 <printf@plt>

18	    a = 3;
   0x00000000000007f0 <+61>:	movl   $0x3,-0x4(%rbp)

19	    b = 4;
   0x00000000000007f7 <+68>:	movl   $0x4,-0x8(%rbp)

20	    usleep(1);
   0x00000000000007fe <+75>:	mov    $0x1,%edi
   0x0000000000000803 <+80>:	callq  0x6b0 <usleep@plt>

21	    memcpy((char *)0x9527, name, sizeof(name));
   0x0000000000000808 <+85>:	mov    0x2007e9(%rip),%rax        # 0x200ff8
   0x000000000000080f <+92>:	mov    (%rax),%rdx
   0x0000000000000812 <+95>:	mov    %rdx,0x9527
   0x000000000000081a <+103>:	mov    0x8(%rax),%rdx
   0x000000000000081e <+107>:	mov    %rdx,0x952f
   0x0000000000000826 <+115>:	mov    0x10(%rax),%rdx
   0x000000000000082a <+119>:	mov    %rdx,0x9537
   0x0000000000000832 <+127>:	mov    0x18(%rax),%rdx
   0x0000000000000836 <+131>:	mov    %rdx,0x953f
   0x000000000000083e <+139>:	mov    0x20(%rax),%rdx
   0x0000000000000842 <+143>:	mov    %rdx,0x9547
   0x000000000000084a <+151>:	mov    0x28(%rax),%rdx
   0x000000000000084e <+155>:	mov    %rdx,0x954f
   0x0000000000000856 <+163>:	mov    0x30(%rax),%rdx
   0x000000000000085a <+167>:	mov    %rdx,0x9557
   0x0000000000000862 <+175>:	mov    0x38(%rax),%rdx
   0x0000000000000866 <+179>:	mov    %rdx,0x955f
   0x000000000000086e <+187>:	mov    0x40(%rax),%rdx
   0x0000000000000872 <+191>:	mov    %rdx,0x9567
   0x000000000000087a <+199>:	movzbl 0x48(%rax),%eax
   0x000000000000087e <+203>:	mov    %al,0x956f

22	
23	    usleep(10);
   0x0000000000000885 <+210>:	mov    $0xa,%edi
---Type <return> to continue, or q <return> to quit---
   0x000000000000088a <+215>:	callq  0x6b0 <usleep@plt>

24	    sum = a + b;
   0x000000000000088f <+220>:	mov    -0x8(%rbp),%eax
   0x0000000000000892 <+223>:	mov    -0x4(%rbp),%edx
   0x0000000000000895 <+226>:	add    %edx,%eax
   0x0000000000000897 <+228>:	mov    %eax,-0xc(%rbp)

25	    a = 0;
   0x000000000000089a <+231>:	movl   $0x0,-0x4(%rbp)

26	    b = 0;
   0x00000000000008a1 <+238>:	movl   $0x0,-0x8(%rbp)

27	    return a+b;
   0x00000000000008a8 <+245>:	mov    -0x8(%rbp),%eax
   0x00000000000008ab <+248>:	mov    -0x4(%rbp),%edx
   0x00000000000008ae <+251>:	add    %edx,%eax

28	}
   0x00000000000008b0 <+253>:	leaveq 
   0x00000000000008b1 <+254>:	retq   

End of assembler dump.
```


结束、、、、


#  **catchsegv**

`catchsegv`专门用于捕获段错误

还是使用上一节的例子还定位问题
```
[root@VM_0_4_centos example_breakInSo]# catchsegv ./main
sum:3
*** Segmentation fault
Register dump:

 RAX: 00007f22f9428040   RBX: 0000000000000000   RCX: 00007f22f86ff7f0
 RDX: 3231333231616161   RSI: 0000000000000000   RDI: 00007ffecfd29eb0
 RBP: 00007ffecfd29ef0   R8 : 0000000000000000   R9 : 00007f22f868727d
 R10: 00007ffecfd29920   R11: 0000000000000246   R12: 0000000000400530
 R13: 00007ffecfd29ff0   R14: 0000000000000000   R15: 0000000000000000
 RSP: 00007ffecfd29ed0

 RIP: 00007f22f9227812   EFLAGS: 00010202

 CS: 0033   FS: 0000   GS: 0000

 Trap: 0000000e   Error: 00000006   OldMask: 00000000   CR2: 00009527

 FPUCW: 0000037f   FPUSW: 00000000   TAG: 00000000
 RIP: 00000000   RDP: 00000000

 ST(0) 0000 0000000000000000   ST(1) 0000 0000000000000000
 ST(2) 0000 0000000000000000   ST(3) 0000 0000000000000000
 ST(4) 0000 0000000000000000   ST(5) 0000 0000000000000000
 ST(6) 0000 0000000000000000   ST(7) 0000 0000000000000000
 mxcsr: 1f80
 XMM0:  00000000000000000000000000000000 XMM1:  00000000000000000000000000000000
 XMM2:  00000000000000000000000000000000 XMM3:  00000000000000000000000000000000
 XMM4:  00000000000000000000000000000000 XMM5:  00000000000000000000000000000000
 XMM6:  00000000000000000000000000000000 XMM7:  00000000000000000000000000000000
 XMM8:  00000000000000000000000000000000 XMM9:  00000000000000000000000000000000
 XMM10: 00000000000000000000000000000000 XMM11: 00000000000000000000000000000000
 XMM12: 00000000000000000000000000000000 XMM13: 00000000000000000000000000000000
 XMM14: 00000000000000000000000000000000 XMM15: 00000000000000000000000000000000

Backtrace:
./libadd.so(_Z3addii+0x5f)[0x7f22f9227812]
/home/yu.tian/stackBreak/example_breakInSo/main.c:10(main)[0x400634]
/lib64/libc.so.6(__libc_start_main+0xf5)[0x7f22f865c505]
??:?(_start)[0x400559]

Memory map:

00400000-00401000 r-xp 00000000 fd:01 922445 /home/yu.tian/stackBreak/example_breakInSo/main
00600000-00601000 r--p 00000000 fd:01 922445 /home/yu.tian/stackBreak/example_breakInSo/main
00601000-00602000 rw-p 00001000 fd:01 922445 /home/yu.tian/stackBreak/example_breakInSo/main
00c7d000-00ca2000 rw-p 00000000 00:00 0 [heap]
7f22f863a000-7f22f87fd000 r-xp 00000000 fd:01 3911 /usr/lib64/libc-2.17.so
7f22f87fd000-7f22f89fd000 ---p 001c3000 fd:01 3911 /usr/lib64/libc-2.17.so
7f22f89fd000-7f22f8a01000 r--p 001c3000 fd:01 3911 /usr/lib64/libc-2.17.so
7f22f8a01000-7f22f8a03000 rw-p 001c7000 fd:01 3911 /usr/lib64/libc-2.17.so
7f22f8a03000-7f22f8a08000 rw-p 00000000 00:00 0
7f22f8a08000-7f22f8a1d000 r-xp 00000000 fd:01 13 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
7f22f8a1d000-7f22f8c1c000 ---p 00015000 fd:01 13 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
7f22f8c1c000-7f22f8c1d000 r--p 00014000 fd:01 13 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
7f22f8c1d000-7f22f8c1e000 rw-p 00015000 fd:01 13 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
7f22f8c1e000-7f22f8d1f000 r-xp 00000000 fd:01 3919 /usr/lib64/libm-2.17.so
7f22f8d1f000-7f22f8f1e000 ---p 00101000 fd:01 3919 /usr/lib64/libm-2.17.so
7f22f8f1e000-7f22f8f1f000 r--p 00100000 fd:01 3919 /usr/lib64/libm-2.17.so
7f22f8f1f000-7f22f8f20000 rw-p 00101000 fd:01 3919 /usr/lib64/libm-2.17.so
7f22f8f20000-7f22f9009000 r-xp 00000000 fd:01 4235 /usr/lib64/libstdc++.so.6.0.19
7f22f9009000-7f22f9208000 ---p 000e9000 fd:01 4235 /usr/lib64/libstdc++.so.6.0.19
7f22f9208000-7f22f9210000 r--p 000e8000 fd:01 4235 /usr/lib64/libstdc++.so.6.0.19
7f22f9210000-7f22f9212000 rw-p 000f0000 fd:01 4235 /usr/lib64/libstdc++.so.6.0.19
7f22f9212000-7f22f9227000 rw-p 00000000 00:00 0
7f22f9227000-7f22f9228000 r-xp 00000000 fd:01 922615 /home/yu.tian/stackBreak/example_breakInSo/libadd.so
7f22f9228000-7f22f9427000 ---p 00001000 fd:01 922615 /home/yu.tian/stackBreak/example_breakInSo/libadd.so
7f22f9427000-7f22f9428000 r--p 00000000 fd:01 922615 /home/yu.tian/stackBreak/example_breakInSo/libadd.so
7f22f9428000-7f22f9429000 rw-p 00001000 fd:01 922615 /home/yu.tian/stackBreak/example_breakInSo/libadd.so
7f22f9429000-7f22f942d000 r-xp 00000000 fd:01 3908 /usr/lib64/libSegFault.so
7f22f942d000-7f22f962c000 ---p 00004000 fd:01 3908 /usr/lib64/libSegFault.so
7f22f962c000-7f22f962d000 r--p 00003000 fd:01 3908 /usr/lib64/libSegFault.so
7f22f962d000-7f22f962e000 rw-p 00004000 fd:01 3908 /usr/lib64/libSegFault.so
7f22f962e000-7f22f9650000 r-xp 00000000 fd:01 3501 /usr/lib64/ld-2.17.so
7f22f983b000-7f22f9840000 rw-p 00000000 00:00 0
7f22f984c000-7f22f984f000 rw-p 00000000 00:00 0
7f22f984f000-7f22f9850000 r--p 00021000 fd:01 3501 /usr/lib64/ld-2.17.so
7f22f9850000-7f22f9851000 rw-p 00022000 fd:01 3501 /usr/lib64/ld-2.17.so
7f22f9851000-7f22f9852000 rw-p 00000000 00:00 0
7ffecfd0a000-7ffecfd2b000 rw-p 00000000 00:00 0 [stack]
7ffecfda8000-7ffecfdaa000 r-xp 00000000 00:00 0 [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0 [vsyscall]
[root@VM_0_4_centos example_breakInSo]# 
```

可见在Backtrace堆栈信息中，列出了出问题的地方，`_Z3addii+0x5f`即代表问题出在add函数的5f位置，从上一个例子来看，add函数基地址`7b3+5f=0x812`，相吻合