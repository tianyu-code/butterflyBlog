---
title: makefile的变量
date: 2020-07-15 12:36:32
tags:
  - makefile
categories:
  - makefile
---


# 目录

+ 变量的基础
+ 变量值替换
+ 变量嵌套
+ 追加变量值
+ override
+ 多行变量
+ 环境变量
+ 目标变量（局部）
+ 模式变量

<!--------more------->


#  **变量的基础**

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


#  **变量值替换**

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

#  **变量嵌套**

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

#  **追加变量值**

我们可以使用“+=”操作符给变量追加值，如：
```makefile
objects = main.o foo.o bar.o utils.o 
objects += another.o 
```
于是，我们的`$(objects)`值变成：`main.o foo.o bar.o utils.o another.o`（another.o被追加进去了）。

如果变量之前没有定义过，那么，`+=`会自动变成`=`

#  **override**

如果有变量是通常make的命令行参数设置的，那么Makefile中对这个变量的赋值会被忽略。如果你想在 Makefile 中设置这类参数的值，那么，你可以使用`override`指示符,其语法是： 

```
override <variable> = <value> 
override <variable> := <value> 
override <variable> += <more text> 
```

#  **多行变量**

还有一种设置变量值的方法是使用`define`关键字。使用`define`关键字设置变量的值可以有换行，这有利于定义一系列的命令（前面我们讲过“命令包”的技术就是利用这个关键字）。 

`define`指示符后面跟的是变量的名字，而重起一行定义变量的值，定义是以`endef`关键字结束。其工作方式和`=`操作符一样。变量的值可以包含函数、命令、文字，或是其它变量。因为命令需要以`Tab`键开头，所以如果你用`define`定义的命令变量中没有以`Tab`键开头，那么make就不会把其认为是命令。

#  **环境变量**

1. make 运行时的系统环境变量可以在make开始运行时被载入到 Makefile 文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了`-e`参数，那么系统环境变量将覆盖Makefile中定义的变量）。

2. 当make嵌套调用时，上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile中。当然默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层 Makefile 传递，则需要使用exprot 关键字来声明。

3. 当然，我并不推荐把许多的变量都定义在系统环境中，这样在我们执行不用的Makefile时，拥有的是同一套系统变量，这可能会带来更多的麻烦。

> 总结：把这个和全局变量和局部变量一样来理解就行了。

#  **目标变量（局部）**

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


#  **模式变量**

在 GNU 的 make 中，还支持模式变量`(Pattern-specific Variable)`，通过上面的目标变量中我们知道，变量可以定义在某个目标上。模式变量的好处就是，我们可以给定一种`“模式”`，可以把变量定义在符合这种模式的所有目标上。 

我们知道，make的“模式”一般是至少含有一个`%`的，所以我们可以以如下方式 给所有以`.o`结尾的目标定义目标变量： 
`%.o : CFLAGS = -g`

> 补充：其实模式规则的来源就是这个模式变量，%取决于目标

