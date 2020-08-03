---
title: makefile的函数
date: 2020-07-15 14:36:32
tags:
  - makefile
categories:
  - makefile
---


# 目录

+ 条件判断
+ 字符串处理函数
+ 文件名操作函数
+ foreach
+ if
+ call
+ origin

<!--------more------->


#  **条件判断**

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

#  **字符串处理函数**

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


#  **文件名操作函数**
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

#  **foreach**

`$(foreach <var>,<list>,<text>) `   

把参数`<list>`中的单词逐一取出放到参数`<var>`所指定的变量中，然后再执行`<text>`所包含的表达式。每一次`<text>`会返回一个字符串，循环过程中，`<text>`的所返回的每个字符串会以空格分隔，最后当整个循环结束时，`<text>`所返回的每个字符串所组成的整个字符串（以空格分隔）将会是`foreach`函数的返回值.

示例
```makefile
names := a b c d 
files := $(foreach n,$(names),$(n).o) 
```

上面的例子中，`$(name)`中的单词会被挨个取出，并存到变量`n`中，`$(n).o`每次根据`$(n)`计算出一个值，这些值以空格分隔，最后作为 foreach 函数的返回值,是`a.o b.o c.o d.o`。 

> 注意，foreach 中的`<var>`参数是一个临时的局部变量，foreach 函数执行完后，参数`<var>`的变量将不在作用，其作用域只在foreach 函数当中。

#  **if**
```makefile
$(if <condition>,<then-part>) 
$(if <condition>,<then-part>,<else-part>) 
```
`<condition>`参数是`if`的表达式，如果其返回的为非空字符串，那么这个表达式就相当于返回真，于是`<then-part>`会被计算，否则`<else-part>`会被计算。

#  **call**

`$(call <expression>,<parm1>,<parm2>,<parm3>...) `

当make执行这个函数时，`<expression>`参数中的变量，如`$(1)，$(2)，$(3)`等，会被参数`<parm1>，<parm2>，<parm3>`依次取代。而<`expression>`的返回值就是call函数的返回值。例如：
```makefile 
reverse = $(1) $(2) 
foo = $(call reverse,a,b) 
```

#  **origin**

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
