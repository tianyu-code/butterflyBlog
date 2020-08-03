---
title: GDB调试之ldd命令
date: 2020-04-04 15:51:38
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, ldd命令
description: 本文介绍ldd命令的基本使用方式

---


# **简介**

ldd命令可查看文件依赖的所有动态库信息

<!-----more-------->

例如
```
[root@VM_0_4_centos studyCode]# ldd main
	linux-vdso.so.1 =>  (0x00007fff7ed8d000)
	libadd.so => ./libadd.so (0x00007fd684f83000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fd684bb5000)
	libstdc++.so.6 => /lib64/libstdc++.so.6 (0x00007fd6848ae000)
	libm.so.6 => /lib64/libm.so.6 (0x00007fd6845ac000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007fd684396000)
/lib64/ld-linux-x86-64.so.2 (0x00007fd685185000)
```

可以看到main依赖的所有动态库，和动态库是否存在（若不存在则=>后会有所提示）