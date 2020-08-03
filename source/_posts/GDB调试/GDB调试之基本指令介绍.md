---
title: GDB调试之基本指令介绍
date: 2020-03-29 21:29:42
tags:
  - GDB调试
categories:
  - GDB
keywords: GDB, 调试,基本指令介绍
description: 本文介绍GDB调试的基本指令
sticky: 100

---

# 目录

+ GDB简介
+ run
+ attach
+ list、search
+ break及相关断点指令
+ catch捕捉
+ step、next、finish、continue控制指令
+ print、wathch、display及相关查看指令
+ backtrace
+ 断点命令
+ 指定源码路径
+ 查看源代码加载内存
+ GDB图形化


#  [GDB简介](http://www.gnu.org/software/gdb/)

> `show language` 可查看当前调试环境语言
`set language`    可查看GDB支持的所有语言种类
`set language <语言>` 可设置当前调试环境语言

<!--------more------->
> What is GDB?    
GDB, the GNU Project debugger, allows you to see what is going on `inside' another program while it executes -- or what another program was doing at the moment it crashed.   
GDB can do four main kinds of things (plus other things in support of these) to help you catch bugs in the act:   
1.Start your program, specifying anything that might affect its behavior.   
2.Make your program stop on specified conditions.   
3.Examine what has happened, when your program has stopped.   
4.Change things in your program, so you can experiment with correcting the effects of one bug and go on to learn about another.   
Those programs might be executing on the same machine as GDB (native), on another machine (remote), or on a simulator. GDB can run on most popular UNIX and Microsoft Windows variants, as well as on Mac OS X.




# **run**

+ 启动被调试的程序，缩写为 r
+ 若程序执行需要参数，可以在run后面直接加入运行参数，或者在run之前执行`set args `

> 备注：stop可暂停程序运行，使用info program可查看程序运行状态即停止原因

# **attach**

 挂接到已在运行的进程来调试 attach \<process-id>

> attach时，再次run，可将原进程杀掉，从头运行程序

> 可以使用pidof 进程名来获得进程ID

# **list**

1. 查看源码list，缩写l
    + 直接输入l为从第一行开始显示，
    + **l   line num** 可显示行号附近的源码
    + **l   function** 可显示函数名称附近的源码
    + **l   filename:linenum** 可显示源码文件filename的linenum附近源码
    + **l   filename:function** 可显示源码文件filename的function函数附近源码
    + 设置一次列出的行数
    ```
    (gdb) set listsize 20
    (gdb) show listsize
    Number of source lines gdb will list by default is 20.
    ```
    + **l first,last**打印first到last行之间的源码，first可缺省，代表当前位置

2. search
    可使用该命令搜索函数、变量等

# **break及相关断点指令**

1. **break**
    在指定的行或函数处设置断点，缩写为 b
    + **break** 当不带参数时，在所选栈帧中执行的下一条指令处设置断点。
    +  **break function-name**在函数体入口处打断点
    +  **break  line-number** 在当前源码文件指定行的开始处打断点。
    +  **break -N** ；**break +N** 在当前源码行前面或后面的 N 行开始处打断点，N 为正整数。
    +  **break  filename:linenum** 在源码文件 filename 的 linenum 行处打断点。
    +  **break  filename:function** 在源码文件 filename 的 function 函数入口处打断点。
    +  **break  address** 在程序指令的地址处打断点，`注意使用时在地址前面加入*`。
    + **break ... if  cond**  设置条件断点，... 代表上述参数之一（或无参数），cond为条件表达式，仅在 cond 值非零时暂停程序执行。

> 若你需要打断点的函数存在若干（函数重载），这是GDB会给你列出一个所有该函数的列表，可自行选择

2.  **tbreak**
    设置临时断点，参数同 break，但在程序第一次停住后会被自动删除
3. **info breakpoints**
    打印未删除的所有断点，观察点和捕获点的列表，缩写为 i b
4. **disable**
    禁用断点，缩写为 dis
5. **clear**
    清除指定行或函数处的断点
6. **delete**
    清除指定num的断点，num为info b中的序号

> 补充：在程序运行时，若没有遇到断点，可直接输入crtl+c停止程序运行，进入GDB命令行

7. **断点条件管理**
    + **condition**  
    用法：
    `condition breakNum expr`修改断点的停止条件为`expr`，
    `condition breakNum `清除断点的停止条件
    + **ignore**
    用法：`ignore breakNUm count`忽略断点count次
    > 上述breakNum均为info b中显示的断点号

8. **线程断点**

    当你的程序是多线程时，你可以定义你的断点是否在所有的线程上，或是在某个特定的线程。
    `break line thread threadNo`
    其中`line`为你的源码行数，threadNo为`info threads`命令中GDB给出的线程ID，若不指定`threadNo`，则为所有线程打断点。

实例如下

```
(gdb) b 27
Breakpoint 1 at 0x400777: file main.c, line 27.
(gdb) b 28
Breakpoint 2 at 0x400785: file main.c, line 28.
(gdb) b 29
Breakpoint 3 at 0x40078f: file main.c, line 29.
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400777 in main at main.c:27
2       breakpoint     keep y   0x0000000000400785 in main at main.c:28
3       breakpoint     keep y   0x000000000040078f in main at main.c:29
(gdb) disable 1
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x0000000000400777 in main at main.c:27
2       breakpoint     keep y   0x0000000000400785 in main at main.c:28
3       breakpoint     keep y   0x000000000040078f in main at main.c:29
(gdb) enable 1
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400777 in main at main.c:27
2       breakpoint     keep y   0x0000000000400785 in main at main.c:28
3       breakpoint     keep y   0x000000000040078f in main at main.c:29
(gdb) clear 27
Deleted breakpoint 1 
(gdb) info b
Num     Type           Disp Enb Address            What
2       breakpoint     keep y   0x0000000000400785 in main at main.c:28
3       breakpoint     keep y   0x000000000040078f in main at main.c:29
```

# **catch捕捉**

可设置捕捉点来补捉程序运行时的一些事件。如：载入共享库（动态链接库）或是C++的异常。设置捕捉点的格式为：
`catch event`
当event发生时，停住程序。event可以是下面的内容：
+ **throw** 一个C++抛出的异常。
+ **catch** 一个C++捕捉到的异常。
+ **exec** 调用系统调用exec时。
+ **fork** 调用系统调用fork时。
+ **vfork** 调用系统调用vfork时。
+ **load** 或 load 载入共享库（动态链接库）时。
+ **unload** 或 unload 卸载共享库（动态链接库）时。

`tcatch event`
只设置一次捕捉点，当程序停住以后，应点被自动删除。

# **step、next、finish、continue控制指令**

1. **step**
    单步跟踪，如果有函数调用，会进入该函数，缩写为 s

2. **next**
    单步跟踪，如果有函数调用，不会进入该函数，缩写为n

> 单步运行加入i参数可单补执行汇编代码

3. **finish**
    执行直到选择的栈帧返回，缩写为 fin(即执行到当前函数返回)

4. **continue**
    当gdb遇到断点停下时，恢复程序执行，缩写为 c



# **print、watch、display及相关查看指令**

1. **print**
    `print/f` ,打印变量或表达式的值，缩写为 p，其中`/f`为可选参数，可指定打印输出格式，并且使用该命令可以**直接给变量或者表达式赋值**

    > 若局部变量和全局变量冲突，或想打印其他文件或函数中的某个变量时，可用
    > file::variable
    > function::variable来指示

    ```
    (gdb) p a
    $9 = 100
    (gdb) p a=10
    $10 = 10
    ```
    
    + (1)打印数组或内存内容
        + `p 数组名称`，若变量本身就是数组，这样可打印数组内容
        + `p *指针@个数`，若变量为动态申请的内存，想要按照指针类型打印内容，可使用该方式

        实例如下：

        ```
        (gdb) p testArray 
        $1 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0}
        (gdb) p pTmp 
        $2 = (int *) 0x602010
        (gdb) p *pTmp@10
        $3 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0}
        ```

    + (2)若字符串过长，或者数组过大，则可以设置print打印的元素个数来显示完全

        `set print elements number-of-elements`，设置显示元素个数
        `show print elements`，查看显示元素个数
        `set print null-stop <on><off>`，如果打开了这个选项，那么显示字符串时则遇到结束符停止显示，默认关闭
        `show print null-stop`
        ```
        (gdb) p testArray 
        $4 = "global_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_te"...
        (gdb) set print elements 1000
        (gdb) p testArray 
        $5 = "global_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_testglobal_array_t"...
        ```
    + (3)设置打印结构体或者类时，将成员分行打印，避免堆积在一起不易查看

        `set print pretty <on><off>`，打开/关闭分行打印
        `show print pretty`，查看当前显示形式
        ```
        (gdb) p testStruct 
        $1 = {para1 = 100, para2 = 200, testArray = "global_array_testglobal_array_testglobal_array_testglobal_array_"}
        (gdb) set print pretty
        (gdb) p testStruct 
        $2 = {
        para1 = 100, 
        para2 = 200, 
        testArray = "global_array_testglobal_array_testglobal_array_testglobal_array_"
        }
        ```
    + (4)设置打印数组时，每个元素单独占一行
        `set print array <on><off>`，打开/关闭元素独占一行的方式
        `show print array`，查看当前显示形式
        ```
        (gdb) p testArray 
        $4 = {1, 2, 3, 4, 5, 6, 7, 8, 9, 0}
        (gdb) set print array on
        (gdb) p testArray 
        $5 =   {1,
        2,
        3,
        4,
        5,
        6,
        7,
        8,
        9,
        0}
        ```
    + (5)设置显示结构体时，是否显示其内的联合体数据
        `set print union <on><off>`，打开/关闭显示其内的联合体数据
        `show print union`，查看当前显示形式

    + (6)查看历史数据
        在每次使用`print`命令时，GDB都会为其输出编号：$1,$2,$3等
        `show value`打印所有历史数据
        `show value n`打印以n为中心的10条历史数据
        `show value +`打印上次显示的历史数据的之后10条历史数据
    
2. **watch**
    + 为表达式（或变量）设置观察点，当表达式（或变量）的值有变化时，暂停程序执行
    + **rwatch expr** 当表达式（变量）expr被读时，停住程序
    + **awatch expr** 当表达式（变量）的值被读或被写时，停住程序。
    + 用法： watch [-l|-location] \<expr>  每当一个表达式的值改变时，观察点就会暂停程序执行

    > 如果给出了 -l 或者 -location，则它会对 expr 求值并观察它所指向的内存。例如，watch *(int *)0x12345678 将在指定的地址处观察一个 4 字节的区域（假设 int 占用 4 个字节）

3. **display**
    `display/fmt <expr><addr>`每次程序停止时打印`expr`表达式或`addr`地址中的内容的值（自动显示）
    `fmt`为显示格式，且支持i参数显示汇编代码

    关于display的相关控制命令：
    + `undisplay <num>`，删除自动显示项
    + `enable <num>`，使能自动显示项
    + `disable <num>`，失能自动显示项（不删除）
    + `info display`，查看display的相关信息，上述三个命令的参数均为info display中列出的`num`

    实例如下

    ```
    (gdb) display a
    1: a = 10
    (gdb) info display
    Auto-display expressions now in effect:
    Num Enb Expression
    1:   y  a
    (gdb) display b
    2: b = 2
    (gdb) info display
    Auto-display expressions now in effect:
    Num Enb Expression
    2:   y  b
    1:   y  a
    (gdb) undisplay 1
    (gdb) info display
    Auto-display expressions now in effect:
    Num Enb Expression
    2:   y  b
    (gdb) c
    Continuing.
    progran is start

    Breakpoint 5, main () at main.c:29
    29	    test();
    2: b = 2
    (gdb) 
    ```

5. **examine**
    查看内存，缩写为x，用法： x/nfu \<addr>

    n、f 和 u 都是可选参数，用于指定要显示的内存以及如何格式化。addr 是要开始显示内存的地址的表达式。

    + **n** 重复次数（默认值是 1），指定要显示多少个单位（由 u 指定）的内存值。

    + **f** 显示格式（初始默认值是 x），显示格式是 print('x'，'d'，'u'，'o'，'t'，'a'，'c'，'f'，'s') 使用的格式之一，再加 i（机器指令）。

    + **u** 单位大小，b 表示单字节，h 表示双字节，w 表示四字节，g 表示八字节。

    例如
    ```
    (gdb) x &a
    0x7fffffffe47c:	0x0000000a
    (gdb) x/d &a
    0x7fffffffe47c:	10
    (gdb) x &b
    0x7fffffffe478:	2
    (gdb) x/4x &b
    0x7fffffffe478:	0x00000002	0x0000000a	0x00000000	0x00000000
    ```

6. **info register**
    查看寄存器的值，其中register可简写为`reg`
    `info register`，查看寄存器（除了浮点寄存器）
    `info all-register`，查看所有寄存器（包括浮点寄存器）
    `info register regName`，查看指定寄存器`regName`的内容


    ```
    (gdb) info reg
    rax            0x4006e7	4196071
    rbx            0x0	0
    rcx            0x400730	4196144
    rdx            0x7fffffffe578	140737488348536
    rsi            0x7fffffffe568	140737488348520
    rdi            0x1	1
    rbp            0x7fffffffe480	0x7fffffffe480
    rsp            0x7fffffffe480	0x7fffffffe480
    r8             0x7ffff7dd5e80	140737351868032
    r9             0x0	0
    r10            0x7fffffffdfa0	140737488347040
    r11            0x7ffff7a2f410	140737348039696
    r12            0x4005b0	4195760
    r13            0x7fffffffe560	140737488348512
    r14            0x0	0
    r15            0x0	0
    rip            0x4006eb	0x4006eb <main+4>
    eflags         0x246	[ PF ZF IF ]
    cs             0x33	51
    ss             0x2b	43
    ds             0x0	0
    es             0x0	0
    fs             0x0	0
    gs             0x0	0
    ```

# **backtrace**

查看程序调用栈的信息，缩写为 bt, 加入full参数,`bt full`可打印出所有的变量信息

+ `backtrace n`  n为正整数，表示之打印栈顶上n层的栈信息
+ `backtrace -n`  -n为负数，表示之打印栈底n层的栈信息
+ `frame n` n为正整数，表示第几层栈，frame可简写为f，使用该命令可切换当前你关注的栈信息
+ `up n` n为正整数，该命令表示向栈的上面移动n层
+ `down n` n为正整数，该命令表示向栈的下面移动n层
+ 上面的命令，都会打印出移动到的栈层的信息。如果你不想让其打出信息。你可以使用这三个命令
    + `select-frame` 对应于 `frame` 命令
    + `up-silently n` 对应于 `up` 命令
    + `down-silently n` 对应于 `down` 命令
+ `info frame`该命令可打印出当前栈的详细信息
    + the address of the frame
    + the address of the next frame down (called by this frame)
    + the address of the next frame up (caller of this frame)
    + the language in which the source code corresponding to this frame is written
    + the address of the frame’s arguments
    + the address of the frame’s local variables
    + the program counter saved in it (the address of execution in the caller frame) which registers were saved in the frame
+ `info args`打印当前栈的arguments
+ `info locals`打印当前函数中的所有局部变量及其值
+ `info catch`打印出当前的函数中的异常处理信息

```C
(gdb) bt
#0  add (a=1, b=2) at add.cpp:6
#1  0x00000000004007d4 in main () at main.c:32
(gdb) bt full
#0  add (a=1, b=2) at add.cpp:6
No locals.
#1  0x00000000004007d4 in main () at main.c:32
        testStruct = {
          para1 = 100, 
          para2 = 200, 
          testArray = "global_array_testglobal_array_testglobal_array_testglobal_array_"
        }
        strArray = "local_array_test"
        a = 10
        b = 1634890337
```

# **断点命令**

我们可以使用GDB提供的`commands`命令来设置停止点的运行命令。当运行的程序在被停止住时，可使其执行指定命令，这很有利行自动化调试。对基于GDB的自动化调试是一个强大的支持,格式如下：

```
commands [breakNum]
... command-list ...
end
```

为断点号breakNum指写一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。
> 注意设置时，先输入commands [breakNum],然后GDB会提示你继续输入命令列表，一行一个命令

实例如下，一图流 ♪(＾∀＾●)ﾉ：

<image src=/images/GDB基本指令介绍/commands介绍.png width="70%">

如果你要清除断点上的命令序列，那么只要简单的执行一下`commands`命令，并直接在打个`end`就行了。

# **指定源码路径**
某些时候，用-g编译过后的执行程序中只是包括了源文件的名字，没有路径名。GDB提供了可以让你指定源文件的路径的命令，以便GDB进行搜索。

> 现在大部分执行程序中都会包括源文件的路径，可简单使用nm -l命令查看，可发现每个符号后都有源文件的路径、名称和行号

> GDB可以在没有源代码的情况下调试，但只能调试汇编代码

`directory dirname ...`使用该命令可指定源文件路径，其中directory可简写为dir
`directory`可清除所有的自定义源文件路径
`show directory`可查看自定义源文件路径

```
(gdb) dir /home
Source directories searched: /home:$cdir:$cwd
(gdb) show dir
Source directories searched: /home:$cdir:$cwd
(gdb) dir
Reinitialize source path to empty? (y or n) y
Source directories searched: $cdir:$cwd
(gdb) show dir
Source directories searched: $cdir:$cwd
```

# **查看源代码加载内存**

其实这里就是查看程序执行时，源码在内存当中的地址
`info line + <line><func><file:line><file:func>`

> 使用disassemble命令可直接查看源码的汇编代码，其中会同时显示加载地址

# **反汇编**

`disassemble <func><'file'::func><start,end><start,+length>`命令可查看汇编代码，具体参数介绍如下

+ `/m`,同时打印源码
+ `/r`,同时打印16进制机器码
+ `func`,打印指定函数的汇编代码
+ `'file'::func`，打印指定文件中的某个函数的汇编代码，格式必须按照前面的写，注意单引号
+ `start,end`，打印start到end地址之间的汇编代码
+ `start,+length`，打印start到start+length之间的汇编代码

原文如下：
```
(gdb) help disas
Disassemble a specified	section	of memory.
Default	is the function	surrounding the	pc of the selected frame.
With a /m modifier, source lines are included (if available).//同时打印源码
With a /r modifier, raw	instructions in	hex are	included.//同时打印16进制机器码
With a single argument,	the function surrounding that address is dumped.
Two arguments (separated by a comma) are taken as a range of memory to dump,
  in the form of "start,end", or "start,+length".

Note that the address is interpreted as	an expression, not as a	location
like in	the "break" command.
So, for	example, if you	want to	disassemble function bar in file foo.c
you must type "disassemble 'foo.c'::bar" and not "disassemble foo.c:bar".
```

# **改变程序的执行**

在GDB中可以根据自己的调试思路来动态地在GDB中更改当前被调试程序的运行线路或是其变量的值

> 博主在学习了函数调用的堆栈变化后，曾尝试在GDB中通过改变寄存器的值，达到改变运行路线的效果，原来GDB直接提供了这个功能
> 具体的尝试见另一篇文章[《GDB调试之改变程序执行流程》](https://kind-ptolemy-135b80.netlify.com/2020/04/06/gdb%E8%B0%83%E8%AF%95%E4%B9%8B%E6%94%B9%E5%8F%98%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B/)

1. 改变变量值
    `print a=100`，使用print命令即可直接改变变量值
2. 跳转执行
    `jump line`，指定下一条语句的运行点
    `jump *address`，指定下一条语句的运行地址

    > 注意，jump命令不会改变当前的程序栈中的内容，所以，当你从一个函数跳到另一个函数时，当函数运行完返回时进行弹栈操作时必然会发生错误,所以最好是同一个函数中进行跳转。
    而博主的那篇文章则是使用改变栈内容达到安全调用和返回的效果的，和GDB的jump还是有些不同的，嘿嘿😁
3. 强制调用函数
    `call <expr>`，强制调用函数expr，该方式能够正常返回
    
    > 实测了一下确实能够正常返回，且在调用的时候压了很多栈空间，不过并没有在里面明显的看到返回的指令地址，具体有待研究

4. 强制函数返回
    `return <expr>`，强制取消当前函数的执行，并立即返回,若指定了`<expr>`则将其作为返回值返回
    
    > 注意：该指令和finish可不一样，finish是正常执行完毕返回，而该指令强制取消执行返回



# **GDB图形化**

在进入gdb时加入指令-tui参数可进入图形界面（Text User Interface）
效果如下，具体操作见另一篇博客[《GDB调试之图形化界面（TUI）》](https://kind-ptolemy-135b80.netlify.com/2020/04/04/gdb%E8%B0%83%E8%AF%95%E4%B9%8B%E5%9B%BE%E5%BD%A2%E5%8C%96%E7%95%8C%E9%9D%A2%EF%BC%88tui%EF%BC%89/)

<image src=/images/GDB_GUI.png width="70%">


> 发现这个界面之后，我只能说

<image src=/images/真香.gif >


