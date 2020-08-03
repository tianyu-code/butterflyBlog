---
title: GDB调试之strip
date: 2020-03-28 23:52:08
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, strip
description: 本文介绍strip命令的使用方法

---


# stri命令简介

> GNU strip discards all symbols from object files objfile.  The list of object files may include archives.  At least one object file must be given.

简单来讲就是给文件脱衣服，包括可执行文件和动态库等。
<image src=/images/你好骚.gif>

> 可使用file命令查看文件的属性，file + 文件名，会显示出是否被strip

例如，我们写一个肥肠简单的代码main.c
```
int add(int a, int b)
{
    return a+b;
}
```
编译生成so后，使用strip生成main_release.so，然后**使用file查看两个so的状态**

```
[root@VM_0_4_centos studyCode]# gcc main.c -o main.so -shared -g -fPIC
[root@VM_0_4_centos studyCode]# ls
main.c  main.so
[root@VM_0_4_centos studyCode]# strip main.so -o main_release.so
[root@VM_0_4_centos studyCode]# ls
main.c  main_release.so  main.so

[root@VM_0_4_centos studyCode]# file main.so
main.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=2500b8ed287f2dff9f7f89b09f5acddf60f8a6c8, not stripped
[root@VM_0_4_centos studyCode]# file main_release.so 
main_release.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=2500b8ed287f2dff9f7f89b09f5acddf60f8a6c8, stripped
```
**未strip的文件较大，strip后的文件较小**

```
[root@VM_0_4_centos studyCode]# ll
total 24
-rw-r--r-- 1 root root   51 Mar 28 23:03 main.c
-rwxr-xr-x 1 root root 6008 Mar 28 23:41 main_release.so
-rwxr-xr-x 1 root root 8784 Mar 28 23:40 main.so
```
未strip的文件使用nm可查看符号，可使用gdb调试，
strip的文件使用nm无法查看符号（加入-D参数可以继续查看），不可gdb调试，**该过程不可逆**。

> 符号表分为两部分，.systab和.dynsym部分，前者主要用于在debug和link中使用，后者主要在运行时使用，strip去掉的是.systab，所以对可执行文件和.so进行strip操作，不影响程序的执行，但是无法对so进行debug,而如果将中间文件.o进行strip操作，则无法完成编译（无法link）

**nm的-D参数含义如下，即显示动态符号而不是普通符号**

```
[root@VM_0_4_centos studyCode]# nm -help
Usage: nm [option(s)] [file(s)]
 List symbols in [file(s)] (a.out by default).
 The options are:
  -a, --debug-syms       Display debugger-only symbols
  -A, --print-file-name  Print name of the input file before every symbol
  -B                     Same as --format=bsd
  -C, --demangle[=STYLE] Decode low-level symbol names into user-level names
                          The STYLE, if specified, can be `auto' (the default),
                          `gnu', `lucid', `arm', `hp', `edg', `gnu-v3', `java'
                          or `gnat'
      --no-demangle      Do not demangle low-level symbol names
  -D, --dynamic          Display dynamic symbols instead of normal symbols
      --defined-only     Display only defined symbols
```

**使用nm查看两个so后的效果如图**
```
[root@VM_0_4_centos studyCode]# nm main.so
0000000000000655 T add
0000000000201028 B __bss_start
0000000000201028 b completed.6355
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000000570 t deregister_tm_clones
00000000000005e0 t __do_global_dtors_aux
0000000000200e00 t __do_global_dtors_aux_fini_array_entry
0000000000200e10 d __dso_handle
0000000000200e18 d _DYNAMIC
0000000000201028 D _edata
0000000000201030 B _end
000000000000066c T _fini
0000000000000620 t frame_dummy


[root@VM_0_4_centos studyCode]# nm main_release.so 
nm: main_release.so: no symbols
```
**加入-D参数,可以发现结果是相同的。**

```
[root@VM_0_4_centos studyCode]# nm -D main.so 
0000000000000655 T add
0000000000201028 B __bss_start
                 w __cxa_finalize
0000000000201028 D _edata
0000000000201030 B _end
000000000000066c T _fini
                 w __gmon_start__
0000000000000520 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 w _Jv_RegisterClasses


[root@VM_0_4_centos studyCode]# nm -D main_release.so 
0000000000000655 T add
0000000000201028 B __bss_start
                 w __cxa_finalize
0000000000201028 D _edata
0000000000201030 B _end
000000000000066c T _fini
                 w __gmon_start__
0000000000000520 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 w _Jv_RegisterClasses
```


# GDB调试strip后的文件




此时对main_release.so进行gdb是无法加载符号表的（在一开始编译时就已经加入了-g参数）

```
[root@VM_0_4_centos studyCode]# gdb main_release.so 
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-115.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/yu.tian/studyCode/main_release.so...Missing separate debuginfo for /home/yu.tian/studyCode/main_release.so
Try: yum --enablerepo='*debug*' install /usr/lib/debug/.build-id/25/00b8ed287f2dff9f7f89b09f5acddf60f8a6c8.debug
(no debugging symbols found)...done.

```

> 既然strip的过程不可逆，那我们如何对strip后的可执行文件或库进行debug？

为了兼顾，既将符号表去掉了，出问题时又能用符号表。**采用符号表和可执行程序分离的方式**

## 制作符号表

```
[root@VM_0_4_centos studyCode]# objcopy --only-keep-debug main.so main.dbg
[root@VM_0_4_centos studyCode]# nm main.dbg
0000000000000655 T add
0000000000201028 B __bss_start
0000000000201028 b completed.6355
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000000570 t deregister_tm_clones
00000000000005e0 t __do_global_dtors_aux
0000000000200e00 t __do_global_dtors_aux_fini_array_entry
0000000000200e10 d __dso_handle
0000000000200e18 b _DYNAMIC
0000000000201028 B _edata
0000000000201030 B _end
000000000000066c T _fini
0000000000000620 t frame_dummy
0000000000200df8 t __frame_dummy_init_array_entry
00000000000006f8 b __FRAME_END__
0000000000201000 b _GLOBAL_OFFSET_TABLE_
```
**发现使用nm main.dbg和main.so查看到的符号相同,至此符号表已经创建成功**

## 添加符号表连接

```
[root@VM_0_4_centos studyCode]# objcopy --add-gnu-debuglink=main.dbg main_release.so 
[root@VM_0_4_centos studyCode]# gdb main_release.so 
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-115.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/yu.tian/studyCode/main_release.so...Reading symbols from /home/yu.tian/studyCode/main.dbg...done.
done.
```
**可以看到此时gdb已经可以正常加载符号表了**


