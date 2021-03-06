---
title: GDB调试之段信息
date: 2020-03-31 23:32:22
tags:
  - GDB
  - 段信息
categories:
  - GDB
keywords: GDB, 调试, 段信息
description: 本文介绍可执行文件的短信息的相关内容

---


# **简介**

为了更好的调试程序，需要对程序编译后产生的库或可执行程序有一定的了解，本文主要介绍一下其中的段信息

# **.bss段**
BSS段（bss segment）通常是指用来存放程序中未初始化的或者初始化为0的全局变量和局部静态变量的一块内存区域。BSS是英文Block Started by Symbol的简称。BSS段属于静态内存分配。

> 默认初始化为0的全局变量，它不占用程序文件的大小，但是占用程序运行时的内存空间,可自行在程序中定义全局数组，查看初始化和不初始化编译出的文件大小。


# **.data段**

数据段（data segment）通常是指用来存放程序中已初始化的全局变量和局部静态变量的一块内存区域，初始化为0的变量出于编译优化的策略还是被保存在BSS段。数据段属于静态内存分配。


# **.rodata段**

该段也叫常量区，用于存放常量数据，ro就是Read Only之意
但是注意并不是所有的常量都是放在常量数据段的，其特殊情况如下：
1. 有些立即数与指令编译在一起直接放在代码段。
2. 对于字符串常量，编译器会去掉重复的常量，让程序的每个字符串常量只有一份。
3. 有些系统中rodata段是多个进程共享的，目的是为了提高空间的利用率


# **.text段**

text段是用于存放程序代码的，编译时确定,只读。

1. 更进一步讲是存放处理器的机器指令，当各个源文件单独编译之后生成目标文件，经连接器链接各个目标文件并解决各个源文件之间函数的引用，

2. 与此同时，还得将所有目标文件中的.text段合在一起，但不是简单的将它们“堆”在一起就完事，还需要处理各个段之间的函数引用问题。

3. 这部分区域的大小在程序运行前就已经确定，并且内存区域通常属于只读, 某些架构也允许代码段为可写，即允许修改程序。

# **.dynstr段**

该段存放调用动态库信息和调用的相关函数

> 编译程序时，若需要链接动态库，需要加入-l库名称和-L 库路径，其实此时去找的是动态库的符号等信息，编译到自己的程序里，等到运行时，会搜索LD_LIBRARY_PATH去寻找库的本身内容

> 显示加载和隐式加载的区别在于，显示更节省资源，只在需要的时候加载动态库的某个函数，省空间，提速度,而隐式就是方便啦，在运行之前全部动态库加载完毕

# **实例**

写一个简单的例子来查看各个段信息

```
#define DEFINE_VALUE "define_test"

char *globalPStr = "global_point_test";
char globalStr[] = "global_array_test";
int gData = 100;
int gNoData;


int main(void)
{
    char *tmpStr = "lcoal_point_test";
    char strArray[] = "local_array_test";
    int a = 10;
    printf("define:%s \n", DEFINE_VALUE);

    return 0;
}

```
编译生成可执行文件main，
输入`nm main`查看符号信息

```
0000000000601068 B __bss_start
0000000000601068 b completed.6355
0000000000601030 D __data_start
0000000000601030 W data_start
0000000000400470 t deregister_tm_clones
00000000004004e0 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
0000000000400608 R __dso_handle
0000000000600e28 d _DYNAMIC
0000000000601068 D _edata
0000000000601070 B _end
00000000004005f4 T _fini
0000000000400500 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400770 r __FRAME_END__
0000000000601064 D gData
0000000000601000 d _GLOBAL_OFFSET_TABLE_
0000000000601040 D globalPStr
0000000000601050 D globalStr
                 w __gmon_start__
000000000060106c B gNoData
000000000040064c r __GNU_EH_FRAME_HDR
00000000004003e0 T _init
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
0000000000400600 R _IO_stdin_used
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
00000000004005f0 T __libc_csu_fini
0000000000400580 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000040052d T main
                 U printf@@GLIBC_2.2.5
00000000004004a0 t register_tm_clones
0000000000400440 T _start
0000000000601068 D __TMC_END__
```
可以发现：
+ gNoData未初始化，在bss段
+ gData，globalStr，globalPStr均初始化，在data段


使用`objdump -s main`查看各个段的详细信息如下，去掉其中一些我们不关注的段信息

```
Contents of section .text:
 400440 31ed4989 d15e4889 e24883e4 f0505449  1.I..^H..H...PTI
 400450 c7c0f005 400048c7 c1800540 0048c7c7  ....@.H....@.H..
 400460 2d054000 e8b7ffff fff4660f 1f440000  -.@.......f..D..
 400470 b86f1060 0055482d 68106000 4883f80e  .o.`.UH-h.`.H...
 400480 4889e577 025dc3b8 00000000 4885c074  H..w.]......H..t
 400490 f45dbf68 106000ff e00f1f80 00000000  .].h.`..........
 4004a0 b8681060 0055482d 68106000 48c1f803  .h.`.UH-h.`.H...
 4004b0 4889e548 89c248c1 ea3f4801 d048d1f8  H..H..H..?H..H..
 4004c0 75025dc3 ba000000 004885d2 74f45d48  u.]......H..t.]H
 4004d0 89c6bf68 106000ff e20f1f80 00000000  ...h.`..........
 4004e0 803d810b 20000075 11554889 e5e87eff  .=.. ..u.UH...~.
 4004f0 ffff5dc6 056e0b20 0001f3c3 0f1f4000  ..]..n. ......@.
 400500 48833d18 09200000 741eb800 00000048  H.=.. ..t......H
 400510 85c07414 55bf200e 60004889 e5ffd05d  ..t.U. .`.H....]
 400520 e97bffff ff0f1f00 e973ffff ff554889  .{.......s...UH.
 400530 e54883ec 2048c745 f8220640 0048b86c  .H.. H.E.".@.H.l
 400540 6f63616c 5f617248 8945e048 b8726179  ocal_arH.E.H.ray
 400550 5f746573 74488945 e8c645f0 00c745f4  _testH.E..E...E.
 400560 0a000000 be330640 00bf3f06 4000b800  .....3.@..?.@...
 400570 000000e8 98feffff b8000000 00c9c390  ................
 400580 41574189 ff415649 89f64155 4989d541  AWA..AVI..AUI..A
 400590 544c8d25 78082000 55488d2d 78082000  TL.%x. .UH.-x. .
 4005a0 534c29e5 31db48c1 fd034883 ec08e82d  SL).1.H...H....-
 4005b0 feffff48 85ed741e 0f1f8400 00000000  ...H..t.........
 4005c0 4c89ea4c 89f64489 ff41ff14 dc4883c3  L..L..D..A...H..
 4005d0 014839eb 75ea4883 c4085b5d 415c415d  .H9.u.H...[]A\A]
 4005e0 415e415f c390662e 0f1f8400 00000000  A^A_..f.........
 4005f0 f3c3                                 ..              
Contents of section .rodata:
 400600 01000200 00000000 00000000 00000000  ................
 400610 676c6f62 616c5f70 6f696e74 5f746573  global_point_tes
 400620 74006c63 6f616c5f 706f696e 745f7465  t.lcoal_point_te
 400630 73740064 6566696e 655f7465 73740064  st.define_test.d
 400640 6566696e 653a2573 200a00             efine:%s ..     
Contents of section .data:
 601030 00000000 00000000 00000000 00000000  ................
 601040 10064000 00000000 00000000 00000000  ..@.............
 601050 676c6f62 616c5f61 72726179 5f746573  global_array_tes
 601060 74000000 64000000                    t...d...        
Contents of section .comment:
 0000 4743433a 2028474e 55292034 2e382e35  GCC: (GNU) 4.8.5
 0010 20323031 35303632 33202852 65642048   20150623 (Red H
 0020 61742034 2e382e35 2d333929 00        at 4.8.5-39).   

```
可以发现：

+ 其中使用char *str方式指向字符串的，字符串均保存在常量区，也就是.rodata
+ 宏定义中的字符串和printf中的格式化输出信息也在.rodata中
+ 而使用char str[]=“xxx”方式的字符串，全局变量保存在.data段，而局部变量则是保存在了.text代码段，在程序运行时直接赋值