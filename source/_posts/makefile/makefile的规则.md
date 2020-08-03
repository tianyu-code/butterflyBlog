---
title: makefile的规则
date: 2020-07-15 11:36:32
tags:
  - makefile
categories:
  - makefile
---


# 目录

+ 简介
+ 通配符
+ 文件搜寻
+ 伪目标
+ 多目标
+ 静态模式
+ 生成依赖关系

<!--------more------->

#  **简介**

规则包含两个部分，一个是依赖关系，一个是生成目标的方法。在 Makefile 中，规则的顺序是很重要的，因为，Makefile中只应该有一个最终目标，其它的目标都是被这个目标所连带出来的，所以一定要让make知道你的最终目标是什么。一般来说，定义在Makefile中的目标可能会有很多，但是第一条规则中的目标将被确立为最终的目标。如果第一条规则中的目标有很多个，那么，第一个目标会成为最终的目标。make所完成的也就是这个目标。

规则举例
```makefile
targets : prerequisites 
command 
```

#  **通配符**

支持 * ,? ,[…]和shell中的含义是相同的:

+ `*` 匹配任意字符0或无数次
+ `?` 匹配任意字符1次
+ `[…]`匹配括号中给出的任意一个字符，而在括号中加入`！`，表示匹配不在括号中给出的字符

#  **文件搜寻**
在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一个路径告诉 make，让make在自动去找。Makefile文件中的特殊变量`VPATH`就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么make就会在当当前目录找不到的情况下，到所指定的目录中去找寻文件了。 

```makefile
VPATH = src:../headers 
```

上面的的定义指定两个目录，“src”和“../headers”，make会按照这个顺序进行搜索。目录由“冒号”分隔。（当然当前目录永远是最高优先搜索的地方） 

另一个设置文件搜索路径的方法是使用 make的`vpath`关键字（注意，它是全小写的），这不是变量，这是一个make的关键字，这和上面提到的那个`VPATH`变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很灵活的功能。它的使用方法有三种： 

1. `vpath <pattern> <directories> `
为符合模式`<pattern>`的文件指定搜索目录`<directories>`。 

2. `vpath <pattern> `
清除符合模式`<pattern>`的文件的搜索目录。 

3. `vpath `
清除所有已被设置好了的文件搜索目录。
 
vapth 使用方法中的`<pattern>`需要包含`%`字符,意思是匹配零或若干字符， 
例如，“%.h”表示所有以“.h”结尾的文件。<pattern>指定了要搜索的文件集，例如： 

```makefile
vpath %.h ../headers 
```

该语句表示，要求 make 在“../headers”目录下搜索所有以`.h`结尾的文件。（如果某文件在当前目录没有找到的话） .

> 补充：经过实测，文件搜寻后，在执行命令时，将源文件加入了相对路径，而生成的中间文件还是在当前目录.


#  **伪目标**

简单来说就是避免文件重名，例如`make clean`只是想要执行下面的makefile命令，并不是要生成clean这个文件，一般来讲我们的all，clean等动作都可以加入伪目标，统一只执行makefile中写好的命令即可,例如：

```
.PHONY: clean
clean:
rm *.o -f
```

#  **多目标**

Makefile 的规则中的目标可以不止一个，其支持多目标，有可能我们的多个目标同时依赖于一个文件，并且其生成的命令大体类似。于是我们就能把其合并起来。

#  **静态模式**

```makefile
<targets ...>: <target-pattern>: <prereq-patterns ...> 
<commands> 
```

例子:

```
objects = foo.o bar.o 

all: $(objects) 
$(objects): %.o: %.c 
$(CC) -c $(CFLAGS) $< -o $@ 

```

上面的例子中，指明了我们的目标从`$(object)` 中获取，`%.o`表明要所有以`.o`结尾的目标，也就是`foo.o bar.o`,而依赖模式`%.c`则取模式`%.o`的`%`，也就是`foo bar`，并为其加下`.c`的后缀，于是我们的依赖目标就是`foo.c bar.c`。

> 备注：这里感觉和模式规则很像，其实把最左边的目标去掉就是模式规则啦，可能这样写比较明显吧

#  **生成依赖关系**

1. 为什么要使用后缀名为.d的依赖文件

	在Makefile中，目标文件的依赖关系需要包含源文件和一系列的头文件,但是一般在我们的Makefile中的依赖关系都是省略头文件的，这就有一个致命的问题，就是头文件修改时，目标文件不会重新生成。

	如果是一个比较大型的工程，我们必需清楚每一个源文件都包含了哪些头文件，并且在加入或删除某些头文件时，也需要一并修改Makefile，这是一个很没有维护性的工作，所以可以使用C/C++编译器的-M选项自动获取源文件中包含的头文件，并生成一个依赖关系,这个依赖关系就保存在.d文件中。

2. 生成方式

<table><tbody>
    <tr>
        <td>选项</td>
        <td>特点</td>
        <td>共同点</td>
    </tr>
    <tr>
        <td bgcolor=#FF8C00>-M</td>
        <td bgcolor=#FF8C00>依赖关系包含标准库</td>
        <td bgcolor=#FF8C00 rowspan="2">默认打开-E参数，使得编译器在预处理结束就停止编译</td>
    </tr>
    <tr>
        <td bgcolor=#FF8C00>-MM</td>
        <td bgcolor=#FF8C00>依赖关系不包含标准库</td>
    </tr>
    <tr>
        <td bgcolor=#9acd32>-MD</td>
        <td bgcolor=#9acd32>依赖关系包含标准库</td>
        <td bgcolor=#9acd32 rowspan="2">不打开-E参数</td>
    </tr>
    <tr>
        <td bgcolor=#9acd32>-MMD</td>
        <td bgcolor=#9acd32>依赖关系不包含标准库</td>
    </tr>
    <tr>
        <td bgcolor=#1E90FF>-MF + fileName</td>
        <td bgcolor=#1E90FF>将依赖关系写入到fileName文件中</td>
        <td bgcolor=#1E90FF></td>
    </tr>
    <tr>
        <td bgcolor=#FFD700>-MT</td>
        <td bgcolor=#FFD700>在生成的依赖文件中，指定规则中的目标</td>
        <td bgcolor=#FFD700></td>
    </tr>
 </table> 
接下来简单介绍几个例子：

1. `gcc -M main.c`

	终端输出
	```
	main.o: main.c defs.h \
	/usr/include/stdio.h \
	/usr/include/features.h \ 			                                         
	/usr/include/sys/cdefs.h /usr/include/gnu/stubs.h \         			
	/usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stddef.h \ 			 
	/usr/include/bits/types.h \
	/usr/include/bits/pthreadtypes.h \ 			
	/usr/include/_G_config.h /usr/include/wchar.h \ 			
	/usr/include/bits/wchar.h /usr/include/gconv.h \ 			
	/usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stdarg.h \ 			
	/usr/include/bits/stdio_lim.h
	```
 
2. `gcc -MM main.c`
	终端输出
	```
	main.o: main.c defs.h
	```

3. `gcc -M -MF main.d main.c`
	则 “-M” 输出的内容就保存在 main.d 文件中了

4. `-MT选项`

	这个最为重要，可以将.d文件本身作为目标加入到依赖文件中，这样就可在头文件更新时，也更新依赖文件
	```
	gcc  main.c -MM -MF main.d   -MT main.d -MT main.o
	$ cat main.d    #查看生成的依赖文件的内容
	main.d main.o: main.c defs.h
	```
	注:依赖规则中 main.d 和 main.o 目标都是通过 “-MT” 选项指定的
