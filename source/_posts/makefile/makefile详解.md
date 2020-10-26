---
title: makefile详解
date: 2020-08-10 15:36:32
tags:
  - makefile
categories:
  - makefile

keywords: Linux,makefile
cover: /images/封面图/make.jfif
top_img: /images/封面图/make.jfif
description: 本文介绍Linux下makefile的使用。
sticky: 0

---


# 简介

##  **概述**

> 该系列笔记参考《跟我一起写makefile》,目前该笔记仅记录工作中有经常遇到的内容，有待补充

在讲述Makefile 之前，还是让我们先来粗略地看一看 Makefile 的规则。
```makefile
target ... : prerequisites ... 
command 
```

+ `target` 也就是一个目标文件，可以是 Object File，也可以是执行文件。还可以是一个 
标签（Label），对于标签这种特性，在后续的“伪目标”章节中会有叙述

+ `prerequisites` 就是要生成那个 `target` 所需要的文件或是目标。 
+ `command` 也就是make需要执行的命令（任意的 Shell 命令）。

这是一个文件的依赖关系，也就是说`target`这一个或多个的目标文件依赖于`prerequisites`中的文件，其生成规则定义在`command`中。说白一点就是说，`prerequisites`中如果有一个以上的文件比`target`文件要新的话，`command`所定义的命令就会被执行。这就是 Makefile 的规则。

> 依赖关系的实质上就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。

在定义好依赖关系后，后续的那一行定义了如何生成目标文件的操作系统命令，一定要以一个`Tab` 键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较`targets`文件和`prerequisites`文件的修改日期，如果`prerequisites`文件的日期要比`targets`文件的日期要新，或者`target`不存在的话，那么make 就会执行后续定义的命令。 


##  **如何工作**

在默认的方式下，也就是我们只输入 make 命令。那么
1. make 会在当前目录下找名字叫“Makefile”或“makefile”的文件。 
2. 如果找到，它会找文件中的第一个目标文件（假设为all），并把这个文件作为最终的目标文件。 
3. 如果 all 文件不存在，或是 all 所依赖的后面的 .o 文件的文件修改时间要比 all这个文件新，那么，他就会执行后面所定义的命令来生成 all 这个文件。 
4. 如果 all 所依赖的.o 文件也不存在，那么 make 会在当前文件中找目标为.o 文件的依赖性，如果找到则再根据那一个规则生成.o 文件.（这有点像一个堆栈的过程） 
5. 当然，你的 C 文件和 H 文件是存在的啦，于是 make 会生成 .o 文件，然后再用 .o 文件生命 make 的终极任务，也就是执行文件 all了。

这就是整个 make 的依赖性，make 会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出并报错，而对于所定义的命令的错误，或是编译不成功，make 根本不理。make 只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。

##  **makefile中有什么**

1. 显式规则
	显式规则说明了，如何生成一个或多的的目标文件。这是由 Makefile 的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。

2. 隐晦规则
	由于make有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写Makefile。

3. 变量定义
	在Makefile中我们定义一系列的变量，变量一般都是字符串，这个有点像C语言中的宏，当 Makefile 被执行时，其中的变量都会被扩展到相应的引用位置上。

4. 文件指示
	其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像 C 语言中的`include`一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译`#if`一样；还有就是定义一个多行的命令。有关这一部分的内容，我会在后续的部分中讲述。

5. 注释
	Makefile中只有行注释，和Shell脚本一样，其注释是用“#”字符，这个就像 C/C++中的“//”一样。如果你要在你的Makefile中使用“#”字符，可以用反斜框进行转义，如：“\#”。

>  Makefile 中的命令，必须要以 Tab 键开始。


##  **文件名**

默认的情况下，make 命令会在当前目录下按顺序找寻文件名为`“GNUmakefile”、“makefile”、“Makefile”`的文件，找到了解释这个文件。在这三个文件名中，最好使用`“Makefile”`这个文件名，因为，这个文件名第一个字符为大写，这样有一种显目的感觉。最好不要用`“GNUmakefile”`，这个文件是GNU的make识别的。有另外一些make只对全小写的“makefile”文件名敏感。
当然，你可以使用别的文件名来书写Makefile，比如：“Make.Linux”，如果要指定特定的Makefile，你可以使用make的`“-f”和“--file”`参数，如：`make -f Make.Linux`。


##  **引用其他的makefile**

在 Makefile 使用`include`关键字可以把别的Makefile包含进来，这很像C语言的#include，被包含的文件会原模原样的放在当前文件的包含位置。include的语法是： 
`include <filename> `
filename可以是当前操作系统 Shell 的文件模式（可以保含路径和通配符）在include前面可以有一些空字符，但是绝不能是Tab键开始。include 和<filename>可以用一个或多个空格隔开。举个例子，你有这样几个Makefile：a.mk、b.mk、c.mk，还有一个文件叫foo.make，以及一个变量$(bar)，其包含了e.mk和f.mk，那么下面的语句： 
```makefile
include foo.make *.mk $(bar) 
```

等价于： 

```makefile
include foo.make a.mk b.mk c.mk e.mk f.mk 
```

make命令开始时，会把找寻`include`所指出的其它Makefile，并把其内容安置在当前的位置。如果文件都没有指定绝对路径或是相对路径的话，make 会在当前目录下首先寻找，如果当前目录下没有找到，那么，make 还会在下面的几个目录下找： 

1. 如果 make 执行时，有“-I”或“--include-dir”参数，那么 make 就会在这个参数所指定的目录下去寻找。 
2. 如果目录<prefix>/include（一般是：/usr/local/bin 或/usr/include）存在的话，make 也会去找。

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成 makefile 的读取，make 会再重试这些没有找 到，或是不能读取的文件，如果还是不行，make 才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”，
如： 

```makefile
-include <filename> 
```

其表示，无论 include 过程中出现什么错误，都不要报错继续执行。和其它版本make兼容的相关命令是sinclude，其作用和这一个是一样的。 


##  **环境变量**

如果你的当前环境中定义了环境变量MAKEFILES，那么make会把这个变量中的值做一个类似于include的动作。这个变量中的值是其它的Makefile，用空格分隔。只是，它和include不同的是，从这个环境变量中引入的 Makefile 的“目标”不会起作用，如果环境变量中定义的文件发现错误，make也会不理。
 
但是在这里我还是建议不要使用这个环境变量，因为只要这个变量一被定义，那么当你使用 make 时，所有的 Makefile 都会受到它的影响，这绝不是你想看到的。在这里提这个事，只是为了告诉大家，也许有时候你的 Makefile 出现了怪事，那么你可以看看当前环境中有没有定义这个变量。


##  **工作方式**

+ 读入所有的 Makefile。 
+ 读入被 include 的其它 Makefile。 
+ 初始化文件中的变量。 
+ 推导隐晦规则，并分析所有规则。 
+ 为所有的目标文件创建依赖关系链。 
+ 根据依赖关系，决定哪些目标要重新生成。 
+ 执行生成命令。 


# makefile的规则

##  **概念**

规则包含两个部分，一个是依赖关系，一个是生成目标的方法。在 Makefile 中，规则的顺序是很重要的，因为，Makefile中只应该有一个最终目标，其它的目标都是被这个目标所连带出来的，所以一定要让make知道你的最终目标是什么。一般来说，定义在Makefile中的目标可能会有很多，但是第一条规则中的目标将被确立为最终的目标。如果第一条规则中的目标有很多个，那么，第一个目标会成为最终的目标。make所完成的也就是这个目标。

规则举例
```makefile
targets : prerequisites 
command 
```

##  **通配符**

支持 * ,? ,[…]和shell中的含义是相同的:

+ `*` 匹配任意字符0或无数次
+ `?` 匹配任意字符1次
+ `[…]`匹配括号中给出的任意一个字符，而在括号中加入`！`，表示匹配不在括号中给出的字符

##  **文件搜寻**
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


##  **伪目标**

简单来说就是避免文件重名，例如`make clean`只是想要执行下面的makefile命令，并不是要生成clean这个文件，一般来讲我们的all，clean等动作都可以加入伪目标，统一只执行makefile中写好的命令即可,例如：

```
.PHONY: clean
clean:
rm *.o -f
```

##  **多目标**

Makefile 的规则中的目标可以不止一个，其支持多目标，有可能我们的多个目标同时依赖于一个文件，并且其生成的命令大体类似。于是我们就能把其合并起来。

##  **静态模式**

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

##  **生成依赖关系**

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


# makefile的命令


##  **显示命令**
1. 通常，make会把其要执行的命令行在命令执行前输出到屏幕上。当我们用“@”字符在命令行前，那么，这个命令将不被make显示出来，最具代表性的例子是，我们用这个功能来像屏幕显示一些信息。如：
	``` 
	@echo 正在编译 XXX 模块...... 
	```
2. 如果 make 执行时，带入 make 参数`-n或--just-print`，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。 

3. 而make参数`-s或--slient`则是全面禁止命令的显示。

##  **命令执行**
当依赖目标新于目标时，也就是当规则的目标需要被更新时，make会一条一条的执行其后的命令。需要注意的是，如果你要让上一条命令的结果应用在下一条命令时，你应该使用分号分隔这两条命令，且写在同一行:

示例一：
``` makefile
exec: 
cd /home/hchen 
pwd
```

示例二：
``` makefile
exec: 
cd /home/hchen; pwd
```

在示例一中cd命令未生效

##  **命令出错**

每当命令运行完后，make会检测每个命令的返回码，如果命令返回成功，那么make会执行下一条命令,有些时候，命令的出错并不表示就是错误的，为了忽略命令的出错，在命令行前加一个`-`,例如：

```makefile
clean: 
	-rm -f *.o 

```

还有一个全局的办法是，给make加上`-i`或是`--ignore-errors`参数，那么Makefile中所有命令都会忽略错误。而如果一个规则是以“.IGNORE”作为目标的，那么这个规则中的所有命令将会忽略错误。这些是不同级别的防止命令出错的方法，你可以根据你的不同喜欢设置。

还有一个要提一下的make的参数的是`-k`或是`--keep-going`，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则。

##  **定义命令包**

如果 Makefile 中出现一些相同命令序列，那么我们可以为这些相同的命令序列定义一个变量。定义这种命令序列的语法以`define`开始，以`endef`结束，例如： 

```
define run-yacc 
yacc $(firstword $^) 
mv y.tab.c $@ 
endef 
```

这里，“run-yacc”是这个命令包的名字，其不要和Makefile中的变量重名，在 `define”和endef`中的两行就是命令序列，调用方式如下：

```
foo.c : foo.y 
$(run-yacc)
```

# makefile的变量


##  **变量的基础**

1. 变量声明和使用
	变量在声明时需要给予初值，而在使用时，需要给在变量名前加上`$`符号，但最好用小括号`()`或是大括号`{}`把变量给包括起来,如果你要使用真实的`$`字符， 那么你需要用`$$`来表示,例如：

	```makefile
	objects = program.o foo.o utils.o 

	program : $(objects) 
	cc -o program $(objects) 

	$(objects) : defs.h 
	```
	变量会在使用它的地方精确地展开，就像 C/C++中的宏一样

2. 在定义变量的值时，我们可以使用其它变量来构造变量的值

	```makefile
	foo = $(bar) 
	bar = $(ugh) 
	ugh = Huh? 
	```

	上面这种用 `=`的方法可是使用后面的变量来定义，而使用`:=`是无法使用后面的变量来定义的，例如

	```makefile
	y := $(x) bar 
	x := foo 
	```
	那么，y 的值是“bar”，而不是“foo bar”。

3. 定义一个空格

	```makefile
	nullstring := 
	space := $(nullstring) # end of the line
	```

	操作符的右边是很难描述一个空格的，这里采用的技术很管用，先用一个Empty变量来标明变量的值开始了，而后面采用“#”注释符来表示变量定义的终止，这样，我们可以定义出其值是一个空格的变量。请注意这里关于`#`的使用，注释符`#`的这种特性值得我们注意，如果我们这样定义一个变量：

	```makefile
	dir := /foo/bar # directory to put the frobs in 
	```
	`dir`这个变量的值是`/foo/bar,后面还跟了4个空格`，如果我们这样使用这样变量来指定别的目录——`$(dir)/file`那么就完蛋了。 


4. 操作符`?=`

	还有一个比较有用的操作符是“?=”，先看示例： 
	```makefile
	FOO ?= bar 
	```

	其含义是，如果`FOO`没有被定义过，那么变量`FOO`的值就是`bar`，如果先前被定义过，那么这条语将什么也不做。


##  **变量值替换**

其格式是:

```makefile
$(var:a=b)
```
其意思是，把变量`var`中所有以`a`字串`结尾`的`a`替换成`b`字串。这里的`结尾`意思是`空格`或是`结束符`,例如：

```makefile
foo := a.o b.o c.o 
bar := $(foo:.o=.c) 
```

结果`$(bar)`的值就是`a.c b.c c.c`。 

这个用法经常在定义OBJS时，把源文件的`.c换成.o（当然用putsubst函数也可以啦）`

另外一种变量替换的技术是以“静态模式”（参见前面章节）定义的，如： 

```makefile
foo := a.o b.o c.o 
bar := $(foo:%.o=%.c) 
```
这依赖于被替换字串中的有相同的模式，模式中必须包含一个`%`字符，这个例子的结果和上面相同。

##  **变量嵌套**

含义：把变量的值再当成变量。

例如：
```makefile
x = y 
y = z 
a := $($(x))
```
在这个例子中，`$(x)`的值是`y`，所以`$($(x))`就是`$(y)`，于是`$(a)`的值就是`z`（注意是`x=y`，而不是`x=$(y)`）。

在这种方式中，或要可以使用多个变量来组成一个变量的名字，然后再取其值，例如：

```makefile
first_second = Hello 
a = first 
b = second 
all = $($a_$b) 
```

这里的`$a_$b`组成了`first_second`，于是，`$(all)`的值就是`Hello`。

##  **追加变量值**

我们可以使用“+=”操作符给变量追加值，如：
```makefile
objects = main.o foo.o bar.o utils.o 
objects += another.o 
```
于是，我们的`$(objects)`值变成：`main.o foo.o bar.o utils.o another.o`（another.o被追加进去了）。

如果变量之前没有定义过，那么，`+=`会自动变成`=`

##  **override**

如果有变量是通常make的命令行参数设置的，那么Makefile中对这个变量的赋值会被忽略。如果你想在 Makefile 中设置这类参数的值，那么，你可以使用`override`指示符,其语法是： 

```
override <variable> = <value> 
override <variable> := <value> 
override <variable> += <more text> 
```

##  **多行变量**

还有一种设置变量值的方法是使用`define`关键字。使用`define`关键字设置变量的值可以有换行，这有利于定义一系列的命令（前面我们讲过“命令包”的技术就是利用这个关键字）。 

`define`指示符后面跟的是变量的名字，而重起一行定义变量的值，定义是以`endef`关键字结束。其工作方式和`=`操作符一样。变量的值可以包含函数、命令、文字，或是其它变量。因为命令需要以`Tab`键开头，所以如果你用`define`定义的命令变量中没有以`Tab`键开头，那么make就不会把其认为是命令。

##  **环境变量**

1. make 运行时的系统环境变量可以在make开始运行时被载入到 Makefile 文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了`-e`参数，那么系统环境变量将覆盖Makefile中定义的变量）。

2. 当make嵌套调用时，上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile中。当然默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层 Makefile 传递，则需要使用exprot 关键字来声明。

3. 当然，我并不推荐把许多的变量都定义在系统环境中，这样在我们执行不用的Makefile时，拥有的是同一套系统变量，这可能会带来更多的麻烦。

> 总结：把这个和全局变量和局部变量一样来理解就行了。

##  **目标变量（局部）**

```makefile
<target ...> : <variable-assignment> 
<target ...> : overide <variable-assignment> 
```

前面的基础变量可理解为全局的，这个就是局部的,例如:

```makefile
prog : CFLAGS = -g 
prog : prog.o foo.o bar.o 
$(CC) $(CFLAGS) prog.o foo.o bar.o 

prog.o : prog.c 
$(CC) $(CFLAGS) prog.c 

foo.o : foo.c 
$(CC) $(CFLAGS) foo.c
 
bar.o : bar.c 
$(CC) $(CFLAGS) bar.c 
```

在这个示例中，不管全局的$(CFLAGS)的值是什么，在prog目标，以及其所引发的所有规则中（prog.o foo.o bar.o 的规则），`$(CFLAGS)的值都是-g`


##  **模式变量**

在 GNU 的 make 中，还支持模式变量`(Pattern-specific Variable)`，通过上面的目标变量中我们知道，变量可以定义在某个目标上。模式变量的好处就是，我们可以给定一种`“模式”`，可以把变量定义在符合这种模式的所有目标上。 

我们知道，make的“模式”一般是至少含有一个`%`的，所以我们可以以如下方式 给所有以`.o`结尾的目标定义目标变量： 
`%.o : CFLAGS = -g`

> 补充：其实模式规则的来源就是这个模式变量，%取决于目标

# makefile的隐含规则


##  **概念**
在我们使用Makefile时，有一些我们会经常使用，而且使用频率非常高的东西，就是在 Makefile 中的“隐含的”，早先约定了的，不需要我们再写出来的规则。 
例如，把`.c`文件编译成`.o`文件这一规则，你根本就不用写出来，make 会自动推导出这种规则，并生成我们需要的`.o`文件。

“隐含规则”会使用一些我们系统变量，我们可以改变这些系统变量的值来定制隐含规则的运行时的参数。如系统变量“CFLAGS”可以控制编译时的编译器参数。


1. 如何使用

	如果要使用隐含规则生成你需要的目标，你所需要做的就是不要写出这个目标的规则。那么make会试图去自动推导产生这个目标的规则和命令，如果 make 可以自动推导生成这个目标的规则和命令，那么这个行为就是隐含规则的自动推导。例如:

	```makefile
	foo : foo.o bar.o 
	cc –o foo foo.o bar.o $(CFLAGS) $(LDFLAGS) 
	```

	我们可以注意到，这个 Makefile 中并没有写下如何生成 `foo.o` 和 `bar.o` 这两目标的规则和命令。因为 make 的“隐含规则”功能会自动为我们自动去推导这两个目标的依赖目标和生成命令。 make 会在自己的“隐含规则”库中寻找可以用的规则，如果找到，那么就会使用。如果找不到就会报错。在上面的那个例子中，make 调用的隐含规则是，把`.o`的目标的依赖文件置成`.c`，并使用 C 的编译命令`cc –c $(CFLAGS) [.c]` 来生成`.o`的目标。

	也就是说，我们完全没有必要写下下面的两条规则：
	```makefile 
	foo.o : foo.c 
	cc –c foo.c $(CFLAGS) 
	bar.o : bar.c 
	cc –c bar.c $(CFLAGS) 
	```

	因为，这已经是“约定”好了的事了，这就是隐含规则。 当然，如果我们为`.o`文件书写了自己的规则，那么 make 就不会自动推导并调用隐含规则，它会按照我们写好的规则忠实地执行。 

2. 隐含规则有优先级

	在 make 的“隐含规则库”中，每一条隐含规则都在库中有其顺序，越靠前的则是越被经常使用的，这会导致我们有些时候即使我们显示地指定了目标依赖，make也不会管。如下面这条规则（没有命令）：
	```makefile 
	foo.o : foo.p 
	```
	依赖文件“foo.p”（Pascal 程序的源文件）有可能变得没有意义。如果目录下存在了`foo.c`文件，那么我们的隐含规则一样会生效，并会通过`foo.c`调用C的编译器生成`foo.o` 文件。因为，在隐含规则中，Pascal 的规则出现在 C 的规则之后，所以，make 找到可以生成 `foo.o`的 C的规则就不再寻找下一条规则了。

	> 如果你确实不希望任何隐含规则推导，那么，你就不要只写出“依赖规则”，而不写命令。（或者使用make的-r参数禁用所有隐含规则），即使是我们指定了“-r”参数，某些隐含规则还是会生效，因为有许多的隐含规则都是使用了“后缀规则”来定义的，所以，只要隐含规则中有“后缀列表”（也就一系统定义在目标.SUFFIXES的依赖目标 ），那么隐含规则就会生效。 默认的后缀列表是：.out,.a, .ln, .o, .c, .cc, .C, .p, .f, .F, .r, .y, .l, .s, .S, .mod, .sym,.def, .h, .info, .dvi, .tex, .texinfo, .texi, .txinfo, .w, .ch .web, .sh, .elc, .el。

3. 常见隐含规则

	|规则名称|格式|内容
	|---|---|---|
	|C程序的隐含规则|<n>.o|“<n>.o”的目标的依赖目标会自动推导为“<n>.c”，并且其生成命令是“$(CC) –c $(CPPFLAGS) $(CFLAGS)”|

	其余的语言的用不到，此处不表

##  **隐含规则的变量**

在隐含规则中的命令中，基本上都是使用了一些预先设置的变量。你可以在你的 makefile 中改变这些变量的值，或是在 make 的命令行中传入这些值，或是在你的环境变量中设置这些值，无论怎么样，只要设置了这些特定的变量，那么其就会对隐含规则起作用。当然，你也可以利用 make 的`-R`或`--no–builtin-variables`参数来取消你所定义的变量对隐含规则的作用。 

下面列出一些常用变量和其对应的参数，即这些变量在makefile中都是预先设定好的

|变量|含义|默认值|对应参数|默认值|
|---|---|---|---|---|
|AR|函数库打包程序（.a静态库）|ar|ARFLAGS|rv|	
|AS|汇编语言编译程序|as|ASFLAGS|空|
|CC|C 语言编译程序|cc|CFLAGS|空|
|CXX|C++语言编译程序|g++|CXXFLAGS|空|
|CPP|C 程序的预处理器（输出是标准输出设备）|$(CC) –E|C 预处理器参数|空|
|YACC|Yacc 文法分析器（针对于 C 程序）|yacc|YFLAGS|空|
|RM|删除文件命令|rm –f|无|无|
||||LDFLAGS（链接器参数）|空|


##  **模式规则**

1. 介绍
	你可以使用模式规则来定义一个隐含规则。一个模式规则就好像一个一般的规则，只是在规则中，目标的定义需要有 `%` 字符,它的意思是表示一个或多个任意字符。在依赖目标中同样可以使用 `%`，只是依赖目标中的 `%` 的取值，取决于其目标。

	模式规则中，至少在规则的目标定义中要包含 `%`，否则，就是一般的规则。目标中的 `%` 定义表示对文件名的匹配，表示长度任意的非空字符串。例如：`%.c`表示以`.c`结尾的文件名（文件名的长度至少为 3），而`s.%.c`则表示以`s.开头，.c结尾`的文件名（文件名的长度至少为 5）。 

	例如有一个模式规则如下：

	```makefile
	%.o : %.c ; <command ......> 

	```

	其含义是，指出了怎么从所有的`.c`文件生成相应的`.o`文件的规则。如果要生成的目标是`a.o b.o`，那么`%c`就是`a.c b.c`。一旦依赖目标中的`%`模式被确定，那么，make 会被要求去匹配当前目录下所有的文件名，一旦找到，make 就会执行规则下的命令，所以，在模式规则中，目标可能会是多个的，如果有模式匹配出多个目标，make 就会产生所有的模式目标，此时，make 关心的是依赖的文件名和生成目标的命令这两件事。

2. 自动化变量

	<table><tbody>
		<tr>
			<td align="center" valign="middle">变量</td>
			<td align="center" valign="middle">说明</td>
		</tr> 
		<tr>
			<td bgcolor=#9acd32>$@</td>
			<td bgcolor=#9acd32>表示规则中的目标文件集。在模式规则中，如果有多个目标，那么，"$@"就是匹配于目标中模式定义的集合</td>
		</tr>
		<tr>
			<td>$%</td>
			<td>仅当目标是函数库文件中，表示规则中的目标成员名。例如，如果一个目标是"foo.a (bar.o)"，那么，"$%"就是"bar.o"，"$@"就是"foo.a"。如果目标不是函数库文件,那么其值为空</td>
		</tr>    
		<tr>
			<td bgcolor=#9acd32>$<</td>
			<td bgcolor=#9acd32>依赖目标中的第一个目标名字。如果依赖目标是以模式（即"%"）定义的，那么"$<"将是符合模式的一系列的文件集。注意，其是一个一个取出来的</td>
		</tr>
		<tr>
			<td>$?</td>
			<td>所有比目标新的依赖目标的集合。以空格分隔</td>
		</tr>
		<tr>
			<td bgcolor=#9acd32>$^</td>
			<td bgcolor=#9acd32>所有的依赖目标的集合。以空格分隔。如果在依赖目标中有多个重复的，那个这个变量会去除重复的依赖目标，只保留一份</td>
		</tr>
		<tr>
			<td>$+</td>
			<td>这个变量很像"$^"，也是所有依赖目标的集合。只是它不去除重复的依赖目标</td>
		</tr>
		<tr>
			<td bgcolor=#9acd32>$*</td>
			<td bgcolor=#9acd32>这个变量表示目标模式中"%"及其之前的部分。如果目标是"dir/a.foo.b"，并且目标的模式是"a.%.b"，那么，"$*"的值就是"dir/a.foo"。这个变量对于构造有关联的文件名是比较有较。如果目标中没有模式的定义，那么"$*"也就不能被推导出，但是，如果目标文件的后缀是 make 所识别的，那么"$*"就是除了后缀的那一部分。例如：如果目标是"foo.c"，因 为".c"是 make 所能识别的后缀名，所以，"$*"的值就是"foo"。这个特性是 GNU make 的，很有可能不兼容于其它版本的 make，所以，你应该尽量避免使用"$*"，除非是在隐含规则或是静态模式中。如果目标中的后缀是 make 所不能识别的，那么"$*"就是空值。</td>
		</tr>
		
	</table> 


##  **老式风格的后缀规则**

后缀规则是一个比较老式的定义隐含规则的方法,后缀规则会被模式规则逐步地取代,因为模式规则更强更清晰。

1. 双后缀规则

	定义了一对后缀：目标文件的后缀和依赖目标（源文件）的后缀，例如：

	```makefile
	".c.o" 
	```

	相当于
	```makefile
	"%o : %c"。
	```

2. 单后缀   
	单后缀规则只定义一个后缀，也就是源文件的后缀,例如：
	`.c`相当于
	```makefile
	`% :%.c`。 
	```

	注意：后缀规则不允许任何的依赖文件，如果有依赖文件的话，那就不是后缀规则，那些后缀统统被认为是文件名，如： 

	```makefile
	.c.o: foo.h 
	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $< 
	```

	这个例子是说，文件`.c.o`依赖于文件`foo.h`，而不是我们想要的这样： 

	```makefile
	%.o: %.c foo.h 
	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $< 
	```

# makefile的函数


##  **条件判断**

条件表达式的语法为：

```makefile
<conditional-directive> 
<text-if-true> 
endif 

以及

<conditional-directive> 
<text-if-true> 
else 
<text-if-false> 
endif
```


其中`conditional-directive`表示条件关键字，这个关键字有四个
```
ifeq (<arg1>, <arg2>) 
ifneq (<arg1>, <arg2>) 

ifdef <variable-name> 
ifndef <variable-name> 
```

特别注意的是，make是在`读取Makefile时`就计算条件表达式的值，并根据条件表达式的值来选择语句，所以最好不要把自动化变量（如$@等）放入条件表达式中，因为自动化变量是在运行时才有的。

> 这个条件判断非常常用，比如在内核驱动里面，用来判断内核版本呐

##  **字符串处理函数**

1. subst

	`$(subst <from>,<to>,<text>) `

	名称：字符串替换函数。   
	功能：把字串`<text>`中的`<from>`字符串替换成`<to>`。   
	返回：函数返回被替换过后的字符串。

	示例:
	```makefile
	$(subst ee,EE,feet on the street)
	```
	把“feet on the street”中的“ee”替换成“EE”，返回结果是“fEEt on the strEEt”。

2. patsubst

	`$(patsubst <pattern>,<replacement>,<text>) `

	名称：模式字符串替换函数。   
	功能：查找`<text>`中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式`<pattern>`，如果匹配的话，则以`<replacement>`替换。这里，`<pattern>`可以包括通配符`%`，表示任意长度的字串。如果`<replacement>`中也包含`%`，那么它的值就是`<pattern>`中的那个`%`所代表的字串。   
	返回：函数返回被替换过后的字符串。 

	示例： 

	```makefile
	$(patsubst %.c,%.o,x.c.c bar.c) 
	```

	把字串“x.c.c bar.c”符合模式`%.c`的单词替换成`%.o`，返回结果是“x.c.o bar.o” 

	> 备注：“$(objects:.o=.c)” 和 “$(patsubst %.o,%.c,$(objects))”是一样的。

3. strip

	`$(strip <string>) `
	名称：去空格函数。    
	功能：去掉`<string>`字串中开头和结尾的空字符。    
	返回：返回被去掉空格的字符串值。 

	示例： 
	```makefile
	$(strip a b c ) 
	```
	把字串“a b c ”去到开头和结尾的空格，结果是“a b c”。 

4. findstring 

	`$(findstring <find>,<in>) `   
	名称：查找字符串函数。    
	功能：在字串`<in>`中查找`<find>`字串。    
	返回：如果找到，那么返回`<find>`，否则返回空字符串。
	
	示例：
	```makefile
	$(findstring a,a b c) 
	$(findstring a,b c) 
	```
	第一个函数返回“a”字符串，第二个返回“”字符串（空字符串）

5. filter 

	`$(filter <pattern...>,<text>) `   
	名称：过滤函数。    
	功能：以`<pattern>`模式过滤`<text>`字符串中的单词，保留符合模式`<pattern>`的单词,可以有多个模式。    
	返回：返回符合模式`<pattern>`的字串。
	
	示例： 
	```makefile
	sources := foo.c bar.c baz.s ugh.h 
	foo: $(sources) 
	cc $(filter %.c %.s,$(sources)) -o foo 
	```
	`$(filter %.c %.s,$(sources))`返回的值是`foo.c bar.c baz.s`。 


6. filter-out 

	`$(filter-out <pattern...>,<text>) `

	名称：反过滤函数。    
	功能：以`<pattern>`模式过滤`<text>`字符串中的单词，去除符合模式`<pattern>`的单词。可以有多个模式。    
	返回：返回不符合模式`<pattern>`的字串。 

	示例： 
	```makefile
	objects=main1.o foo.o main2.o bar.o 
	mains=main1.o main2.o 
	```
	`$(filter-out $(mains),$(objects))` 返回值是`foo.o bar.o`。

7. sort 

	`$(sort <list>)`   
	名称：排序函数。    
	功能：给字符串`<list>`中的单词排序（升序）。    
	返回：返回排序后的字符串。 

	示例：`$(sort foo bar lose)`返回`bar foo lose`”。 
	> 备注：sort函数会去掉<list>中相同的单词。 

8. word 

	`$(word <n>,<text>)`    
	名称：取单词函数。      
	功能：取字符串`<text>`中第`<n>`个单词。（从1开始）    
	返回：返回字符串`<text>`中第`<n>`个单词。如果`<n>`比`<text>`中的单词数要大，那么返回空字符串。 

	示例：`$(word 2, foo bar baz)`返回值是`bar`。 

9. wordlist 

	`$(wordlist <s>,<e>,<text>)`   
	名称：取单词串函数。    
	功能：从字符串`<text>`中取从`<s>`开始到`<e>`的单词串。`<s>和<e>`是一个数字。    
	返回：返回字符串`<text>`中从`<s>`到`<e>`的单词字串。如果`<s>`比`<text>`中的单词数要大，那么返回空字符串。如果`<e>`大于`<text>`的单词数，那么返回从`<s>`开始，到`<text>`结束的单词串。    

	示例： `$(wordlist 2, 3, foo bar baz)`返回值是`bar baz`。 

10. words 

	`$(words <text>)`    
	名称：单词个数统计函数。    
	功能：统计`<text>`中字符串中的单词个数。    
	返回：返回`<text>`中的单词数。 

	示例：`$(words, foo bar baz)`返回值是`3`。
	
	> 备注：如果我们要取`<text>`中最后的一个单词，我们可以这样：`$(word $(words <text>),<text>)`。 

11. firstword 

	`$(firstword <text>)`    
	名称：首单词函数。    
	功能：取字符串`<text>`中的第一个单词。    
	返回：返回字符串`<text>`的第一个单词。 

	示例：`$(firstword foo bar)`返回值是`foo`。  


12. 字符串函数实例 
	以上，是所有的字符串操作函数，如果搭配混合使用，可以完成比较复杂的功能。这里举一个现实中应用的例子。我们知道make使用`VPATH`变量来指定`依赖文件`的搜索路径。我们可以利用这个搜索路径来指定编译器对头文件的搜索路径参数`CFLAGS`，如：

	```makefile 
	override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH))) 
	```

	如果我们的`$(VPATH)`值是`src:../headers`，那么`$(patsubst %,-I%,$(subst :, ,$(VPATH)))`将返回`-Isrc -I../headers`，这正是gcc搜索头文件路径的参数。


##  **文件名操作函数**
1. dir 

	`$(dir <names...>) `   
	名称：取目录函数。    
	功能：从文件名序列`<names>`中取出目录部分。目录部分是指最后一个反斜杠（“/”）之前的部分。如果没有反斜杠，那么返回“./”。    
	返回：返回文件名序列`<names>`的目录部分。 

	示例： `$(dir src/foo.c hacks)`返回值是`src/ ./`。 


2. notdir 

	`$(notdir <names...>)`    
	名称：取文件函数。    
	功能：从文件名序列`<names>`中取出非目录部分。非目录部分是指最后一个反斜杠（“ /”）之后的部分。    
	返回：返回文件名序列`<names>`的非目录部分。 

	示例： `$(notdir src/foo.c hacks)`返回值是`foo.c hacks`。 


3. suffix 

	`$(suffix <names...>) `   
	名称：取后缀函数。    
	功能：从文件名序列`<names>`中取出各个文件名的后缀。    
	返回：返回文件名序列`<names>`的后缀序列，如果文件没有后缀，则返回空字串。 

	示例：`$(suffix src/foo.c src-1.0/bar.c hacks)`返回值是`.c .c`。 

4. basename 
	`$(basename <names...>) `   
	名称：取前缀函数。    
	功能：从文件名序列`<names>`中取出各个文件名的前缀部分。    
	返回：返回文件名序列`<names>`的前缀序列，如果文件没有前缀，则返回空字串。 

	示例：`$(basename src/foo.c src-1.0/bar.c hacks)`返回值是`src/foo src-1.0/bar hacks`。


5. addsuffix 

	`$(addsuffix <suffix>,<names...>)`    
	名称：加后缀函数。    
	功能：把后缀`<suffix>`加到`<names>`中的每个单词后面。    
	返回：返回加过后缀的文件名序列。 

	示例：`$(addsuffix .c,foo bar)`返回值是`foo.c bar.c`。 

6. addprefix 

	`$(addprefix <prefix>,<names...>)`    
	名称：加前缀函数。    
	功能：把前缀`<prefix>`加到`<names>`中的每个单词后面。    
	返回：返回加过前缀的文件名序列。 

	示例：`$(addprefix src/,foo bar)`返回值是`src/foo src/bar`。


7. join 

	`$(join <list1>,<list2>)`    
	名称：连接函数.    
	功能：把`<list2>`中的单词对应地加到`<list1>`的单词后面。如果`<list1>`的单词个数要比`<list2>`的多，那么`<list1>`中的多出来的单词将保持原样。如果`<list2>`的单词个数要比`<list1>`多，那么，`<list2>`多出来的单词将被复制到`<list1>`中。   
	返回：返回连接过后的字符串。 

	示例：`$(join aaa bbb , 111 222 333)`返回值是`aaa111 bbb222 333`。

##  **foreach**

`$(foreach <var>,<list>,<text>) `   

把参数`<list>`中的单词逐一取出放到参数`<var>`所指定的变量中，然后再执行`<text>`所包含的表达式。每一次`<text>`会返回一个字符串，循环过程中，`<text>`的所返回的每个字符串会以空格分隔，最后当整个循环结束时，`<text>`所返回的每个字符串所组成的整个字符串（以空格分隔）将会是`foreach`函数的返回值.

示例
```makefile
names := a b c d 
files := $(foreach n,$(names),$(n).o) 
```

上面的例子中，`$(name)`中的单词会被挨个取出，并存到变量`n`中，`$(n).o`每次根据`$(n)`计算出一个值，这些值以空格分隔，最后作为 foreach 函数的返回值,是`a.o b.o c.o d.o`。 

> 注意，foreach 中的`<var>`参数是一个临时的局部变量，foreach 函数执行完后，参数`<var>`的变量将不在作用，其作用域只在foreach 函数当中。

##  **if**
```makefile
$(if <condition>,<then-part>) 
$(if <condition>,<then-part>,<else-part>) 
```
`<condition>`参数是`if`的表达式，如果其返回的为非空字符串，那么这个表达式就相当于返回真，于是`<then-part>`会被计算，否则`<else-part>`会被计算。

##  **call**

`$(call <expression>,<parm1>,<parm2>,<parm3>...) `

当make执行这个函数时，`<expression>`参数中的变量，如`$(1)，$(2)，$(3)`等，会被参数`<parm1>，<parm2>，<parm3>`依次取代。而<`expression>`的返回值就是call函数的返回值。例如：
```makefile 
reverse = $(1) $(2) 
foo = $(call reverse,a,b) 
```

##  **origin**

origin函数不像其它的函数，他并不操作变量的值，他只是告诉你你的这个变量是哪里来的,其语法是：
```makefile 
$(origin <variable>) 
```

> 注意，`<variable>`是变量的名字，不应该是引用。所以你最好不要在`<variable>`中使用`$`字符。

Origin函数会以其返回值来告诉你这个变量的`“出生情况”`，下面是origin函数的返回值: 

+ `undefined` : 如果`<variable>`从来没有定义过，origin函数返回这个值 

+ `default` : 如果`<variable>`是一个默认的定义，比如“CC”这个变量，这种变量我们将在后面讲述。

+ `environment`: 如果`<variable>`是一个环境变量，并且当Makefile被执行时`-e`参数没有被打开。 

+ `file` : 如果`<variable>`这个变量被定义在Makefile中。 

+ `command line `: 如果`<variable>`这个变量是被命令行定义的。 

+ `override` : 如果`<variable>`是被`override`指示符重新定义的。 

+ `automatic` : 如果`<variable>`是一个命令运行中的自动化变量。关于自动化变量将在后面讲述。 

这些信息对于我们编写Makefile 非常有用的，例如，环境变量中有一个bletch，而我们的makefile中也 有一个变量“bletch”，此时，我们想判断一下，如果变量来源于环境，那么我们就把它重定义了，如果来源于命令行等非环境的，那么我们就不重新定义它。于 是，在我们的Makefile中可以这样写： 
```makefile
ifdef bletch 
ifeq "$(origin bletch)" "environment" 
bletch = barf, gag, etc. 
endif 
endif 
```

当然，你也许会说，使用`override`关键字不就可以重新定义环境中的变量了吗？为什么需要使用这样的步骤？是的，我们用 `override` 是可以达到这样的效果，可是它过于粗暴，它同时会把从命令行定义的变量也覆盖了，而我们只想重新定义环境传来的，而不想重新定义命令行传来的。


# makefile的参数


##  **指定目标文件**

1. 指定终极目标的方法可以很方便地让我们编译我们的程序，例如下面这个例子： 
    ```makefile
    .PHONY: all 
    all: prog1 prog2 prog3 prog4 
    ```

    从这个例子中，我们可以看到，这个makefile中有四个需要编译的程序——`prog1,prog2,prog3,prog4`，我们可以使用`make all`命令来编译所有的目标（如果把all置成第一个目标，那么只需执行“make”），我们也可以使用“make prog2”来单独编译目标“prog2”。 

2. 即然make可以指定所有makefile中的目标，那么也包括“伪目标”，于是我们可以根据这种性质来让我们的makefile根据指定的不同的目标来完成不同的事。在 Unix 世界中，软件发布时，特别是 GNU 这种开源软件的发布时，其 makefile 都包含了编译、安装、打包等功能。我们可以参照这种规则来书写我们的 makefile 中的目标。 

    + `all`  : 这个伪目标是所有目标的目标，其功能一般是编译所有的目标。 
    + `clean` : 这个伪目标功能是删除所有被 make 创建的文件。 
    + `install` :这个伪目标功能是安装已编译好的程序，其实就是把目标执行文件拷贝到指定的目标中去。 
    + `print` : 这个伪目标的功能是例出改变过的源文件。 
    + `tar` : 这个伪目标功能是把源程序打包备份。也就是一个 tar 文件。 
    + `dist` : 这个伪目标功能是创建一个压缩文件，一般是把 tar 文件压成 Z 文件。或是 gz 文件。 
    + `TAGS` : 这个伪目标功能是更新所有的目标，以备完整地重编译使用。 
    + `check和test` : 这两个伪目标一般用来测试 makefile 的流程。 


3. 在这里我们说一个环境变量，`MAKECMDGOALS`,它保存了你所指定的终极目标的列表，若在命令行中没有指定目标，那么这个值为空，可以用在如下例子中
    ```makefile
    sources = foo.c bar.c 
    ifneq ( $(MAKECMDGOALS),clean) 
    include $(sources:.c=.d) 
    endif 
    ```
    基于上面的这个例子，只要我们输入的命令不是`make clean`，那 makefile会自动包含`foo.d`和`bar.d`这两个文件。

##  **make的参数**


<table><tbody>
    <tr>
        <td>参数缩写</td>
        <td>参数</td>
        <td>作用</td>
    </tr>
    <tr>
        <td>-b<br>-m</td>
        <td> </td>
        <td>忽略和其它版本 make 的兼容性</td>
    </tr>
    <tr>
        <td>-B</td>
        <td>--always-make</td>
        <td>认为所有的目标都需要更新（重编译）</td>
    </tr>
    <tr>
        <td bgcolor=#9acd32>-C (dir)</td>
        <td bgcolor=#9acd32>--directory=(dir)</td>
        <td bgcolor=#9acd32>指定读取 makefile 的目录dir</td>
    </tr>
    <tr>
        <td rowspan="7">—debug[=(options)]</td>
        <td> </td>
        <td>输出 make 的调试信息。它有几种不同的级别可供选择，如果没有参数，那就是输出最简单的调试信息。下面是options的取值：</td>
    </tr>
    <tr>
        <td>a</td>
        <td>也就是all，输出所有的调试信息（会非常的多）</td>
    </tr>
    <tr>
        <td>b</td>
        <td>也就是 basic，只输出简单的调试信息。即输出不需要重编译的目标</td>
    </tr>
    <tr>
        <td>v</td>
        <td>也就是 verbose，在 b 选项的级别之上。输出的信息包括哪个 makefile 被解析，不需要被重编译的依赖文件（或是依赖目标）等</td>
    </tr>
    <tr>
        <td>i</td>
        <td>也就是 implicit，输出所以的隐含规则</td>
    </tr>
    <tr>
        <td>j </td>
        <td>也就是 jobs，输出执行规则中命令的详细信息，如命令的 PID、返回码等</td>
    </tr>
        <tr>
        <td>m</td>
        <td>也就是 makefile，输出 make 读取 makefile，更新 makefile，执行 makefile 的信息</td>
    </tr>
    <tr>
        <td>-d</td>
        <td> </td>
        <td>相当于“--debug=a”</td>
    </tr>
    <tr>
        <td bgcolor=#9acd32>-e</td>
        <td bgcolor=#9acd32>--environment-overrides</td>
        <td bgcolor=#9acd32>指明环境变量的值覆盖 makefile 中定义的变量的值</td>
    </tr>
    <tr>
        <td bgcolor=#9acd32>-f=(file)</td>
        <td bgcolor=#9acd32>--file=(file)<br>--makefile=(file)</td>
        <td bgcolor=#9acd32>指定需要执行的 makefile</td>
    </tr>
    <tr>
        <td>-h</td>
        <td>--help</td>
        <td>显示帮助信息</td>
    </tr>
    <tr>
        <td>-i</td>
        <td>--ignore-errors</td>
        <td>在执行时忽略所有的错误</td>
    </tr>
    <tr>
        <td>-I (dir)</td>
        <td>--include-dir=(dir)</td>
        <td>指定一个被包含 makefile 的搜索目标。可以使用多个“-I”参数来指定多个目录</td>
    </tr>
    <tr>
        <td>-j [(jobsnum)]</td>
        <td>--jobs[=(jobsnum)]</td>
        <td>指同时运行命令的个数。如果没有这个参数，make 运行命令时能运行多少就运行多少</td>
    </tr>
    <tr>
        <td>-k</td>
        <td>--keep-going</td>
        <td>出错也不停止运行。如果生成一个目标失败了，那么依赖于其上的目标就不会被执行了</td>
    </tr>
    <tr>
        <td>-l load</td>
        <td>--load-average[=load]<br>—max-load[=load]</td>
        <td>指定 make 运行命令的负载</td>
    </tr>
    <tr>
        <td bgcolor=#9acd32>-n</td>
        <td bgcolor=#9acd32>--just-print<br>--dry-run<br>--recon</td>
        <td bgcolor=#9acd32>仅输出执行过程中的命令序列，但并不执行</td>
    </tr>
    <tr>
        <td>-o file</td>
        <td>--old-file=file<br>--assume-old=file</td>
        <td>不重新生成的指定的file，即使这个目标的依赖文件新于它。</td>
    </tr>
    <tr>
        <td>-p</td>
        <td>--print-data-base</td>
        <td>输出 makefile 中的所有数据，包括所有的规则和变量。这个参数会让一个简单的makefile都会输出一堆信息。</td>
    </tr>
    <tr>
        <td>-q</td>
        <td>--question</td>
        <td>不运行命令，也不输出。仅仅是检查所指定的目标是否需要更新。如果是 0 则说明要更新，如果是 2 则说明有错误发生。</td>
    </tr>    
    <tr>
        <td>-r</td>
        <td>--no-builtin-rules</td>
        <td>禁止 make 使用任何隐含规则</td>
    </tr>
    <tr>
        <td>-R</td>
        <td>--no-builtin-variabes</td>
        <td>禁止 make 使用任何作用于变量上的隐含规则。</td>
    </tr>    
    <tr>
        <td bgcolor=#9acd32>-s</td>
        <td bgcolor=#9acd32>--silent<br>--quiet</td>
        <td bgcolor=#9acd32>在命令运行时不输出命令的输出</td>
    </tr>
    <tr>
        <td>-S</td>
        <td>--no-keep-going<br>--stop</td>
        <td>取消“-k”选项的作用。因为有些时候，make 的选项是从环境变量“MAKEFLAGS”中继承下来的。所以你可以在命令行中使用这个参数来让环境变量中的“-k”选项失效</td>
    </tr>    
    <tr>
        <td>-t</td>
        <td>--touch</td>
        <td>相当于 UNIX 的 touch 命令，只是把目标的修改日期变成最新的，也就是阻止生成目标的命令运行</td>
    </tr>   
    <tr>
        <td>-v</td>
        <td>--version</td>
        <td>输出 make 程序的版本、版权等关于 make 的信息</td>
    </tr>    
    <tr>
        <td>-w</td>
        <td>--print-directory</td>
        <td>输出运行 makefile 之前和之后的信息。这个参数对于跟踪嵌套式调用 make 时很有用</td>
    </tr>
    <tr>
        <td>--no-print-directory</td>
        <td> </td>
        <td>禁止“-w”选项</td>
    </tr>    
    <tr>
        <td>-W file</td>
        <td>--what-if= file<br>--new-if= file<br>--assume-if= file</td>
        <td>假定目标file需要更新，如果和“-n”选项使用，那么这个参数会输出该目标更新时的运 行动作。如果没有“-n”那么就像运行 UNIX 的“touch”命令一样，使得file的修改时间为当前时间。</td>
    </tr>   
    <tr>
        <td>--warn-undefined-variables</td>
        <td> </td>
        <td>只要 make 发现有未定义的变量，那么就输出警告信息</td>
    </tr>  
</table> 
















---