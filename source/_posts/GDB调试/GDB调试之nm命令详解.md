---
title: GDB调试之nm命令详解
date: 2020-04-01 00:30:41
tags:
  - nm命令
  - GDB
categories:
  - GDB
keywords: GDB, 调试, nm
description: 本文介绍nm命令的基本使用方法
sticky: 100
---


# **简介**

nm命令主要是用来列出某些文件中的符号


# **常用参数介绍**

```
[root@VM_0_4_centos studyCode]# nm -help
Usage: nm [option(s)] [file(s)]
 List symbols in [file(s)] (a.out by default).
 The options are:
  -a,   //实际输出调试信息+正常信息
  -A,   //新增一列显示目标文件名，没有实际意义（即使你的符号是动态库的，也会显示当前目标文件）
  -C,   // 可将低级符号转换成便于用户查看的方式（备注：可将c++的符号转换为class::func的形式）
  -D,   //显示动态符号，即显示so中增加的符号信息和so引用的so的符号信息
  -f,   //改变该命令的输出格式，默认bsd，可选sysv和posix（个人角色sysv还挺好看的）
                
  -g,   //仅显示外部符号，实测和-D参数几乎一样（两者都不会显示–fvisibility=hidden后的符号）
  -l,   //显示符号的行号，附加一列显示xx文件的xx行定义的该行号
  -n,   //将符号按照地址排序，排序后的输出较为整洁
  -r,   //逆序输出符号
  -S,   //输出符号大小，注意是十六进制
  -u,   //仅显示未定义符号，可查看引用其余动态库的符号
  ```

# **效果展示**

## **-f参数的效果**
```
[root@VM_0_4_centos studyCode]# nm main -f sysv


Symbols from main:

Name                  Value           Class        Type         Size             Line  Section

add                 |                |   U  |              FUNC|                |     |*UND*
__bss_start         |0000000000601078|   B  |            NOTYPE|                |     |.bss
completed.6355      |0000000000601078|   b  |            OBJECT|0000000000000001|     |.bss
__data_start        |0000000000601040|   D  |            NOTYPE|                |     |.data
data_start          |0000000000601040|   W  |            NOTYPE|                |     |.data
deregister_tm_clones|0000000000400580|   t  |              FUNC|                |     |.text
__do_global_dtors_aux|00000000004005f0|   t  |              FUNC|                |     |.text
__do_global_dtors_aux_fini_array_entry|0000000000600e08|   t  |            OBJECT|                |     |.fini_array
__dso_handle        |0000000000400738|   R  |            OBJECT|                |     |.rodata
_DYNAMIC            |0000000000600e18|   d  |            OBJECT|                |     |.dynamic
_edata              |0000000000601078|   D  |            NOTYPE|                |     |.data
_end                |0000000000601080|   B  |            NOTYPE|                |     |.bss
_fini               |0000000000400724|   T  |              FUNC|                |     |.fini
frame_dummy         |0000000000400610|   t  |              FUNC|                |     |.text
__frame_dummy_init_array_entry|0000000000600e00|   t  |            OBJECT|                |     |.init_array
__FRAME_END__       |00000000004008a0|   r  |            OBJECT|                |     |.eh_frame
gData               |0000000000601074|   D  |            OBJECT|0000000000000004|     |.data
_GLOBAL_OFFSET_TABLE_|0000000000601000|   d  |            OBJECT|                |     |.got.plt
globalPStr          |0000000000601050|   D  |            OBJECT|0000000000000008|     |.data
globalStr           |0000000000601060|   D  |            OBJECT|0000000000000012|     |.data
__gmon_start__      |                |   w  |            NOTYPE|                |     |*UND*
gNoData             |000000000060107c|   B  |            OBJECT|0000000000000004|     |.bss
__GNU_EH_FRAME_HDR  |000000000040077c|   r  |            NOTYPE|                |     |.eh_frame_hdr
_init               |00000000004004e0|   T  |              FUNC|                |     |.init
__init_array_end    |0000000000600e08|   t  |            NOTYPE|                |     |.init_array
__init_array_start  |0000000000600e00|   t  |            NOTYPE|                |     |.init_array
_IO_stdin_used      |0000000000400730|   R  |            OBJECT|0000000000000004|     |.rodata
__JCR_END__         |0000000000600e10|   d  |            OBJECT|                |     |.jcr
__JCR_LIST__        |0000000000600e10|   d  |            OBJECT|                |     |.jcr
__libc_csu_fini     |0000000000400720|   T  |              FUNC|0000000000000002|     |.text
__libc_csu_init     |00000000004006b0|   T  |              FUNC|0000000000000065|     |.text
__libc_start_main@@GLIBC_2.2.5|                |   U  |              FUNC|                |     |*UND*
main                |000000000040063d|   T  |              FUNC|0000000000000064|     |.text
printf@@GLIBC_2.2.5 |                |   U  |              FUNC|                |     |*UND*
register_tm_clones  |00000000004005b0|   t  |              FUNC|                |     |.text
_start              |0000000000400550|   T  |              FUNC|                |     |.text
__TMC_END__         |0000000000601078|   D  |            OBJECT|                |     |.data
```

## **-l参数的效果**

```
0000000000601078 B __bss_start
0000000000601078 b completed.6355
0000000000601040 D __data_start
0000000000601040 W data_start
0000000000400580 t deregister_tm_clones
00000000004005f0 t __do_global_dtors_aux
0000000000600e08 t __do_global_dtors_aux_fini_array_entry
0000000000400738 R __dso_handle
0000000000600e18 d _DYNAMIC
0000000000601078 D _edata
0000000000601080 B _end
0000000000400724 T _fini
0000000000400610 t frame_dummy
0000000000600e00 t __frame_dummy_init_array_entry
00000000004008a0 r __FRAME_END__
0000000000601074 D gData	/home/yu.tian/studyCode/main.c:10
0000000000601000 d _GLOBAL_OFFSET_TABLE_
0000000000601050 D globalPStr	/home/yu.tian/studyCode/main.c:8
0000000000601060 D globalStr	/home/yu.tian/studyCode/main.c:9
                 w __gmon_start__
000000000060107c B gNoData	/home/yu.tian/studyCode/main.c:11
000000000040077c r __GNU_EH_FRAME_HDR
00000000004004e0 T _init
0000000000600e08 t __init_array_end
0000000000600e00 t __init_array_start
0000000000400730 R _IO_stdin_used
0000000000600e10 d __JCR_END__
0000000000600e10 d __JCR_LIST__
0000000000400720 T __libc_csu_fini
00000000004006b0 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000040063d T main	/home/yu.tian/studyCode/main.c:14
                 U printf@@GLIBC_2.2.5
00000000004005b0 t register_tm_clones
0000000000400550 T _start
0000000000601078 D __TMC_END__

```

## **-n参数的效果**

```
[root@VM_0_4_centos studyCode]# nm main -n
                 U add
                 w __gmon_start__
                 U __libc_start_main@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
00000000004004e0 T _init
0000000000400550 T _start
0000000000400580 t deregister_tm_clones
00000000004005b0 t register_tm_clones
00000000004005f0 t __do_global_dtors_aux
0000000000400610 t frame_dummy
000000000040063d T main
00000000004006b0 T __libc_csu_init
0000000000400720 T __libc_csu_fini
0000000000400724 T _fini
0000000000400730 R _IO_stdin_used
0000000000400738 R __dso_handle
000000000040077c r __GNU_EH_FRAME_HDR
00000000004008a0 r __FRAME_END__
0000000000600e00 t __frame_dummy_init_array_entry
0000000000600e00 t __init_array_start
0000000000600e08 t __do_global_dtors_aux_fini_array_entry
0000000000600e08 t __init_array_end
0000000000600e10 d __JCR_END__
0000000000600e10 d __JCR_LIST__
0000000000600e18 d _DYNAMIC
0000000000601000 d _GLOBAL_OFFSET_TABLE_
0000000000601040 D __data_start
0000000000601040 W data_start
0000000000601050 D globalPStr
0000000000601060 D globalStr
0000000000601074 D gData
0000000000601078 B __bss_start
0000000000601078 b completed.6355
0000000000601078 D _edata
0000000000601078 D __TMC_END__
000000000060107c B gNoData
0000000000601080 B _end
```

## **-C参数的效果之于C++函数**

不加参数效果
```
[root@VM_0_4_centos studyCode]# nm libadd.so 
0000000000201028 B __bss_start
0000000000201028 b completed.6355
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000000580 t deregister_tm_clones
00000000000005f0 t __do_global_dtors_aux
0000000000200e00 t __do_global_dtors_aux_fini_array_entry
0000000000200e10 d __dso_handle
0000000000200e18 d _DYNAMIC
0000000000201028 D _edata
0000000000201030 B _end
000000000000067c T _fini
0000000000000630 t frame_dummy
0000000000200df8 t __frame_dummy_init_array_entry
0000000000000708 r __FRAME_END__
0000000000201000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000688 r __GNU_EH_FRAME_HDR
0000000000000528 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000200e08 d __JCR_END__
0000000000200e08 d __JCR_LIST__
                 w _Jv_RegisterClasses
00000000000005b0 t register_tm_clones
0000000000201028 d __TMC_END__
0000000000000665 T _Z3addii
```
加入-C参数效果
```
0000000000201028 B __bss_start
0000000000201028 b completed.6355
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000000580 t deregister_tm_clones
00000000000005f0 t __do_global_dtors_aux
0000000000200e00 t __do_global_dtors_aux_fini_array_entry
0000000000200e10 d __dso_handle
0000000000200e18 d _DYNAMIC
0000000000201028 D _edata
0000000000201030 B _end
000000000000067c T _fini
0000000000000630 t frame_dummy
0000000000200df8 t __frame_dummy_init_array_entry
0000000000000708 r __FRAME_END__
0000000000201000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000688 r __GNU_EH_FRAME_HDR
0000000000000528 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000200e08 d __JCR_END__
0000000000200e08 d __JCR_LIST__
                 w _Jv_RegisterClasses
00000000000005b0 t register_tm_clones
0000000000201028 d __TMC_END__
0000000000000665 T add(int, int)
```
可以发现add这个函数，由于c++加入的一些前缀，变成了参数
## **-u和-D参数的对比**

```
[root@VM_0_4_centos studyCode]# nm main -u
                 U add
                 w __gmon_start__
                 U __libc_start_main@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
[root@VM_0_4_centos studyCode]# nm main -D
                 U add
0000000000601078 B __bss_start
0000000000601078 D _edata
0000000000601080 B _end
0000000000400724 T _fini
                 w __gmon_start__
00000000004004e0 T _init
                 U __libc_start_main
                 U printf
```

-u参数列出了本文件中使用但没有定义的符号，即引用的其他库的符号