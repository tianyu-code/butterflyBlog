---
title: GDB调试之objdump命令
date: 2020-04-04 15:18:37
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, objdump
description: 本文介绍objdump命令的详细使用方法和示例
sticky: 0
---

# **简介**

objdump命令主要是用来查看文件中的各个段的详细信息

# **常用参数介绍**

```C
[root@VM_0_4_centos studyCode]# objdump --help
Usage: objdump <option(s)> <file(s)>
 Display information from object <file(s)>.
 At least one of the following switches must be given:
    -a,   //显示档案库的成员信息,类似ls -l将lib*.a的信息列出。
    -f,   //显示objfile中每个文件的整体头部摘要信息。
    -p,   //显示特定文件头内容（可看到该文件依赖哪些动态库和GLIBC版本）
    -h,   //显示目标文件各个section的头部摘要信息，包括各个段的位置，大小等
    -x,   //显示所有的头部信息，实测基本包含了-a，-f，-p和-h参数的输出
    -C，  //可将低级符号转换成便于用户查看的方式，可用于C++的函数显示，和nm命令的-C一样
    -d,   //从objfile中反汇编那些特定指令机器码的section
    -D,   //与 -d 类似，但反汇编所有section. 
    -S,   //反汇编且带源码
    -s,   //显示所有section的完整内容
    -g,   //显示调试信息。企图解析保存在文件中的调试信息并以C语言的语法显示出来。仅仅支持某些类型的调试信息。有些其他的格式被readelf -w支持
    -e,   //类似-g选项，但是生成的信息是和ctags工具相兼容的格式

    //使用-t和-T参数，objdump也能像nm命令一样查看符号表
    -t,   //显示符号表
    -T,   //显示动态符号表

    -r,   //显示重定位信息
    -R,   //显示动态链接重定位信息

    -j,   //仅仅显示指定名称为name的section的信息

    //指定目标文件的小端。这个项将影响反汇编出来的指令。在反汇编的文件没描述小端信息的时候用。
    -EB --endian=big               
    -EL --endian=little            

    -l,   //用文件名和行号标注相应的目标代码
    -F,   //显示在文件中的偏移地址

    //这两个参数可选定显示的开始地址和结束地址，注意ADDR要加上0x，变成16进制，可搭配-s，-d等参数使用
    --start-address=ADDR       //开始地址
    --stop-address=ADDR        //结束地址

```

比较常用的参数有：`-C，-s，-j，-S，-l --start(stop)-address=ADDR`,使用这几个参数即可查看段信息和反汇编


# **应用场景**

> 博主由于经历有限，仅对以下使用场景有所领悟

一个简单的例子，源码如下
```C
#include "add.h"

int add(int a, int b)
{
   // usleep(1);
    return a+b;
}

```
编译生成动态库

1. **想要查看指定函数的汇编源码时，有两种方式**

    + 使用`nm | grep`快速查找该函数的地址
    ```
    [root@VM_0_4_centos studyCode]# nm -C libadd.so | grep add
    0000000000000685 T add(int, int)
    ```

    获得地址0x685后，然后使用`objdump -S -l --start-address=XXXX`的方式可以直接显示XXX地址开始的汇编代码和源代码（注意XXXX部分的地址前面需要加上0x表示16进制）
    ```
    [root@VM_0_4_centos studyCode]# objdump libadd.so -S -l -C --start-address=0x685

    libadd.so:     file format elf64-x86-64


    Disassembly of section .text:

    0000000000000685 <add(int, int)>:
    _Z3addii():
    /home/yu.tian/studyCode/add.cpp:4
    #include "add.h"

    int add(int a, int b)
    {
    685:	55                   	push   %rbp
    686:	48 89 e5             	mov    %rsp,%rbp
    689:	89 7d fc             	mov    %edi,-0x4(%rbp)
    68c:	89 75 f8             	mov    %esi,-0x8(%rbp)
    /home/yu.tian/studyCode/add.cpp:6
    // usleep(1);
        return a+b;
    68f:	8b 45 f8             	mov    -0x8(%rbp),%eax
    692:	8b 55 fc             	mov    -0x4(%rbp),%edx
    695:	01 d0                	add    %edx,%eax
    /home/yu.tian/studyCode/add.cpp:7
    }
    697:	5d                   	pop    %rbp
    698:	c3                   	retq   

    Disassembly of section .fini:

    000000000000069c <_fini>:
    _fini():
    69c:	48 83 ec 08          	sub    $0x8,%rsp
    6a0:	48 83 c4 08          	add    $0x8,%rsp
    6a4:	c3                   	retq   

    ```
    即可获得指定函数的汇编代码

    + 第二种方式为进入**GDB**使用**disassemble**命令反汇编，（`disassemble/m`可同时显示源码）

    ```C
    [root@VM_0_4_centos studyCode]# gdb libadd.so 
    GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-115.el7
    Copyright (C) 2013 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-redhat-linux-gnu".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>...
    Reading symbols from /home/yu.tian/studyCode/libadd.so...done.
    (gdb) disas/m add
    Dump of assembler code for function add(int, int):
    4	{
    0x0000000000000685 <+0>:	push   %rbp
    0x0000000000000686 <+1>:	mov    %rsp,%rbp
    0x0000000000000689 <+4>:	mov    %edi,-0x4(%rbp)
    0x000000000000068c <+7>:	mov    %esi,-0x8(%rbp)

    5	   // usleep(1);
    6	    return a+b;
    0x000000000000068f <+10>:	mov    -0x8(%rbp),%eax
    0x0000000000000692 <+13>:	mov    -0x4(%rbp),%edx
    0x0000000000000695 <+16>:	add    %edx,%eax

    7	}
    0x0000000000000697 <+18>:	pop    %rbp
    0x0000000000000698 <+19>:	retq   

    End of assembler dump.

    ```
2. **查看指定段信息**

例如我们想要查看一个动态库的代码段，可如下操作
```C
[root@VM_0_4_centos studyCode]# objdump libadd.so -s -j .text

libadd.so:     file format elf64-x86-64

Contents of section .text:
 05a0 488d0588 0a200048 8d3d7a0a 20005548  H.... .H.=z. .UH
 05b0 29f84889 e54883f8 0e77025d c3488b05  ).H..H...w.].H..
 05c0 240a2000 4885c074 f25dffe0 0f1f4000  $. .H..t.]....@.
 05d0 488d0551 0a200048 8d3d4a0a 20005548  H..Q. .H.=J. .UH
 05e0 29f84889 e548c1f8 034889c2 48c1ea3f  ).H..H...H..H..?
 05f0 4801d048 d1f87502 5dc3488b 15ef0920  H..H..u.].H.... 
 0600 004885d2 74f25d48 89c6ffe2 0f1f4000  .H..t.]H......@.
 0610 803d110a 20000075 2748833d d7092000  .=.. ..u'H.=.. .
 0620 00554889 e5740c48 8d3db207 2000e85d  .UH..t.H.=.. ..]
 0630 ffffffe8 68ffffff 5dc605e8 09200001  ....h...].... ..
 0640 f3c30f1f 4000662e 0f1f8400 00000000  ....@.f.........
 0650 48833d80 07200000 7426488b 057f0920  H.=.. ..t&H.... 
 0660 004885c0 741a5548 8d3d6a07 20004889  .H..t.UH.=j. .H.
 0670 e5ffd05d e957ffff ff0f1f80 00000000  ...].W..........
 0680 e94bffff ff554889 e5897dfc 8975f88b  .K...UH...}..u..
 0690 45f88b55 fc01d05d c3                 E..U...].
```

当然也可以搭配start/stop address来显示指定地址的信息，效果如下

```
[root@VM_0_4_centos studyCode]# objdump libadd.so -s -j .text --start-address=0x600 --stop-address=0x670

libadd.so:     file format elf64-x86-64

Contents of section .text:
 0600 004885d2 74f25d48 89c6ffe2 0f1f4000  .H..t.]H......@.
 0610 803d110a 20000075 2748833d d7092000  .=.. ..u'H.=.. .
 0620 00554889 e5740c48 8d3db207 2000e85d  .UH..t.H.=.. ..]
 0630 ffffffe8 68ffffff 5dc605e8 09200001  ....h...].... ..
 0640 f3c30f1f 4000662e 0f1f8400 00000000  ....@.f.........
 0650 48833d80 07200000 7426488b 057f0920  H.=.. ..t&H.... 
 0660 004885c0 741a5548 8d3d6a07 20004889  .H..t.UH.=j. .H.
```








