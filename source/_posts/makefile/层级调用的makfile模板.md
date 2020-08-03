---
title: 层级调用的makefile模板
date: 2020-07-14 20:36:32
tags:
  - makefile
categories:
  - makefile
---


# 目录

+ 简介
+ 公共部分makefile
+ 模块目录makefile
+ 顶层makefile
+ 实际输出


<!--------more------->

#  **简介**

学习完《跟我一起写makefile》之后，对于makefile的各种规则和函数有了初步了解，但是对于层级调用的makefile不是很熟悉，接下来的内容是搜索的网上的一个模板，对其中内容自己做了一些修改，并且简单实验了一下可以正常工作.这个模板与其他的不相同的是，他把公共部分封装成了一个makefile，这个思路感觉很有意思。

原文链接  [《一个适用于层级目录结构的makefile模版》](https://www.cnblogs.com/coderkian/p/3479564.html)

本次makefile工程的目录内容如下

<img src=/images/层级调用makefile/目录结构.png width="40%">


#  **公共部分makefile**

首先来看看公共部分的makefile，该makefile将各个makefile公用的部分整个起来，节省了很多工作，具体解释见其中注释


```makefile
####################  Makefile.env  #######################
# Top level pattern, include by Makefile of child directory
# in which variable like TOPDIR, TARGET or LIB may be needed
###########################################################

CC=gcc
MAKE=make

AR=ar cr

CFLAGS=-Wall -g -O0

#获取下一层目录名称,以便后续循环调用下一层目录makefile
#其中exclude_dirs代表不需要进行检索的目录，该变量由顶层makefile给出
dirs:=$(shell find . -maxdepth 1 -type d)
dirs:=$(basename $(patsubst ./%,%,$(dirs)))
dirs:=$(filter-out $(exclude_dirs),$(dirs))
SUBDIRS:=$(dirs)

#这里的目的是将主模块的路径放在最后，以便其余子模块先编译
#基本原理是先给出主模块的目录，在检索到下一层级包含有主模块目录时，
#将主模块目录先从其中过滤出来，然后在拼接到末尾
SOURCEDIR=main
TMPDIR:=$(filter $(SOURCEDIR),$(dirs))

ifeq ($(TMPDIR),$(SOURCEDIR))
SUBDIRS:=$(filter-out $(SOURCEDIR),$(dirs)) $(SOURCEDIR)
endif

#获取源文件，目标文件和依赖文件名
SRCS=$(wildcard *.c)
OBJS=$(patsubst %.c,%.o,$(SRCS))
DEPS=$(patsubst %.c,%.d,$(SRCS))

################ all #################
#这里的TARGET，LIB变量分别由主模块和子模块的makefile给出，若无则不进行规则
.PHONY=all
all:$(TARGET) $(LIB) subdirs
$(LIB):$(OBJS)
	@echo "$(shell pwd) TARGET=$(TARGET) LIB=$(LIB) SUBDIRS=$(SUBDIRS)"
	$(AR) $@ $^
	cp $@ $(LIBPATH)
	cp $(INCLUDE) $(INCLUDEPATH)

#该规则是这个makefile的重点，代表层级调用，循环调用每个子目录下的makefile
subdirs:$(SUBDIRS)
	@echo "$(shell pwd) TARGET=$(TARGET) LIB=$(LIB) SUBDIRS=$(SUBDIRS)"
	for dir in $(SUBDIRS);\
	do $(MAKE) -C $$dir all||exit 1;\
	done

$(TARGET):$(OBJS)
	@echo "$(shell pwd) TARGET=$(TARGET) LIB=$(LIB) SUBDIRS=$(SUBDIRS)"
	@echo "Compiling $(TARGET)"
	$(CC) $^ -o $@ $(LDFLAGS) -L $(LIBPATH)
	cp $(TARGET) $(EXEPATH)

%.o:%.c
	@echo "Compiling $<"
	$(CC) -c  $< -o $@ $(CFLAGS) -I $(INCLUDEPATH)

###########  生成和包含依赖文件include .d  ###############
#这里表示当make clean时就不进行这个操作了
ifneq ($(MAKECMDGOALS),clean)
include $(DEPS)
endif

#注意：这里人为指定了依赖关系中的目标为.o和.d，为的是在h文件更新时，依赖文件也能更新
%.d:%.c
	@echo "creating $@"
	$(CC) $< -MM -MF $@ -MT $*.o -MT $*.d -I $(INCLUDEPATH)

########### clean ###################
.PHONY=clean
clean:
	@echo "$(shell pwd) cleaning "
	for dir in $(SUBDIRS);\
	do $(MAKE) -C $$dir clean||exit 1;\
	done
	rm -f $(OBJS) $(TARGET) $(LIB) $(DEPS)

```

其实这个公共部分的makefile，主要的地方就是all的依赖，除了subdirs，LIB和TARGET都是不存在的，所以后面的规则也就不会执行，subdirs的规则主要是调用下一层目录的makefile。而在下一层的makefiel中将LIB或者TARGET传入，用来编译各自的内容。

#  **模块目录makefile**

1. src目录的makefile

    这个makefile很简单，由于该目录下无源文件，继续调用下一层makefile即可。

2. add目录makefile
    ```makefile
      TOPDIR=./../..

      LIB=libadd.a

      INCLUDE=add.h
      INCLUDEPATH=$(TOPDIR)/include
      LIBPATH=$(TOPDIR)/lib

      include $(TOPDIR)/Makefile.env
    ```
  
    这里提供了LIB变量，那么在公共部分的makefile中的LIB的规则就会执行啦，另外还传入了一些规则中用到的变量.

3. main目录的makefile
    ```makefile
    TOPDIR=./../..

    TARGET=main

    LIBPATH=$(TOPDIR)/lib
    EXEPATH=$(TOPDIR)/bin
    INCLUDEPATH=$(TOPDIR)/include
    LDFLAGS= -ladd

    include $(TOPDIR)/Makefile.env
    ```
    这里提供了TARGET变量，那么在公共部分的makefile中的TARGET的规则就会执行，另外还传入了一些规则中用到的变量



#  **顶层makefile**

```makefile
TOPDIR=./

exclude_dirs= include bin lib 
export exclude_dirs

.PHONY=all
all:
	make -f $(TOPDIR)/Makefile.env all

.PHONY=clean
clean:
	make -f $(TOPDIR)/Makefile.env clean
	-rm -f include/* bin/* lib/*

```

在这个makefile中，给出了不需要进行检索的目录变量exclude_dirs，另外这里的变量使用了export，是为了在后续调用的makefile中都存在这个变量，因为是使用make -f调用的，而上面的几个makefile是使用include直接引用的，所以不存在这个问题。


#  **实际输出**

```
[root@VM_0_4_centos qwe]# make
make -f .//Makefile.env all
make[1]: Entering directory `/home/yu.tian/qwe'
/home/yu.tian/qwe TARGET= LIB= SUBDIRS=src
for dir in src;\
do make -C $dir all||exit 1;\
done
make[2]: Entering directory `/home/yu.tian/qwe/src'
/home/yu.tian/qwe/src TARGET= LIB= SUBDIRS=add main
for dir in add main;\
do make -C $dir all||exit 1;\
done
make[3]: Entering directory `/home/yu.tian/qwe/src/add'
../../Makefile.env:58: add.d: No such file or directory
creating add.d
gcc add.c -MM -MF add.d -MT add.o -MT add.d -I ./../../include
make[3]: Leaving directory `/home/yu.tian/qwe/src/add'
make[3]: Entering directory `/home/yu.tian/qwe/src/add'
Compiling add.c
gcc -c  add.c -o add.o -Wall -I ./../../include
/home/yu.tian/qwe/src/add TARGET= LIB=libadd.a SUBDIRS=
ar cr libadd.a add.o
cp libadd.a ./../../lib
cp add.h ./../../include
/home/yu.tian/qwe/src/add TARGET= LIB=libadd.a SUBDIRS=
for dir in ;\
do make -C $dir all||exit 1;\
done
make[3]: Leaving directory `/home/yu.tian/qwe/src/add'
make[3]: Entering directory `/home/yu.tian/qwe/src/main'
../../Makefile.env:58: main.d: No such file or directory
creating main.d
gcc main.c -MM -MF main.d -MT main.o -MT main.d -I ./../../include
make[3]: Leaving directory `/home/yu.tian/qwe/src/main'
make[3]: Entering directory `/home/yu.tian/qwe/src/main'
Compiling main.c
gcc -c  main.c -o main.o -Wall -I ./../../include
/home/yu.tian/qwe/src/main TARGET=main LIB= SUBDIRS=
Compiling main
gcc main.o -o main -ladd -L ./../../lib
cp main ./../../bin
/home/yu.tian/qwe/src/main TARGET=main LIB= SUBDIRS=
for dir in ;\
do make -C $dir all||exit 1;\
done
make[3]: Leaving directory `/home/yu.tian/qwe/src/main'
make[2]: Leaving directory `/home/yu.tian/qwe/src'
make[1]: Leaving directory `/home/yu.tian/qwe'
```

编译过程基本和我们构想的一致，并且该makefile无论修改任何一个模块的.c或.h文件，都会重新编译涉及的文件。