---
title: gcc的参数
date: 2020-07-15 16:47:32
tags:
  - makefile
  - gcc
categories:
  - makefile
---


# 目录

+ 常用选项
+ 预处理选项
+ 警告选项

<!--------more------->


#  **常用选项**
<table><tbody>
    <tr>
        <td align="center" valign="middle">选项</td>
        <td align="center" valign="middle">作用</td>
    </tr>
    <tr>
        <td  align="center" colspan="2" bgcolor=#1E90FF>总体选项</td>
    </tr>   
    <tr>
        <td>-c</td>
        <td>编译或汇编源文件，但是不作连接.编译器输出对应于源文件的目标文件。GCC默认用`.o'替换源文件名后缀`.c'，`.i'，`.s'等等。GCC 忽略-c 选项后面任何无法识别的输入文件</td>
    </tr>
    <tr>
        <td>-S</td>
        <td>编译后即停止,不进行汇编。生成.s汇编源文件</td>
    </tr>    
    <tr>
        <td>-E</td>
        <td>预处理后即停止,不进行编译</td>
    </tr>
    <tr>
        <td> -o file</td>
        <td>指定输出文件为file</td>
    </tr>
    <tr>
        <td align="center" valign="middle" colspan="2" bgcolor=#1E90FF>链接选项</td>
    </tr> 
    <tr>
        <td>-llibrary</td>
        <td>链接名为 library 的库文件，链接器在标准搜索目录中寻找这个库文件,库文件的真正名字`liblibrary.a'</td>
    </tr>
    <tr>
        <td>-shared</td>
        <td>生成一个共享目标文件,他可以和其他目标文件连接产生可执行文件</td>
    </tr>
    <tr>
        <td>-Wl,option</td>
        <td>把选项 option 传递给连接器.如果 option 中含有逗号,就在逗号处分割成多个选项.这个选项可以用于给动态库传入SO－NAME，格式为-Wl，-soname, MYLIB.so</td>
    </tr>
    <tr>
        <td>-Idir</td>
        <td>在头文件的搜索路径列表中添加 dir 目录.</td>
    </tr>
    <tr>
        <td>-Ldir</td>
        <td>在`-l'选项的搜索路径列表中添加 dir 目录</td>
    </tr>
    <tr>
        <td>-static</td>
        <td>默认情况下， GCC在链接时优先使用动态链接库，只有当动态链接库不存在时才考虑使用静态链接库，如果需要的话可以在编译时加上-static选项，强制使用静态链接库</td>
    </tr> 
    <tr>
        <td align="center" valign="middle" colspan="2" bgcolor=#1E90FF>调试选项</td>
    </tr>
    <tr>
        <td>-g(level)</td>
        <td>以操作系统的本地格式(stabs, COFF, XCOFF,或 DWARF).产生调试信息. GDB 能够使用这些调试信息.-g可以指定输出的调试信息的等级，默认2级，最多-g3</td>
    </tr>
    <tr>
        <td>-O0</td>
        <td>不进行代码优化</td>
    </tr>
    <tr>
        <td>-O1<br>-O2<br>-O3</td>
        <td>不同等级优化，优化我们暂时不介绍<br>1.优化可能对调试带来问题，任何级别的优化都将带来代码结构的改变，例如对分支的合并和消除<br>2.内存操作顺序改变带来的问题，也就是volatile关键字起作用的时候啦</td>
    </tr>
    <tr>
        <td align="center" valign="middle" colspan="2" bgcolor=#1E90FF>代码生成</td>
    </tr>
    <tr>
        <td>-fomit-frame-pointer</td>
        <td>取消帧指针，即不使用ebp，而是使用esp直接计算帧上的变量的位置，好处是可以多出一个ebp寄存器，但是坏处却很多，比如帧上寻址变慢，且无法调试，尽量不使用这个参数
</td>
    </tr> 
    <tr>
        <td>-fPIC</td>
        <td>生成地址无关代码，一般用于动态库，如果可执行文件是动态链接的，那GCC默认会使用PIC来产生可执行文件的代码段</td>
    </tr>
</table> 

#  **预处理选项**
 
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

#  **警告选项**

警告选项我们简单的例举了几个，不过我们使用的时候，使用`-Wall`就可以打开几乎所有常用的警告啦，`-Werror`代表将警告视为错误，这个不包含在`-Wall`中，需要单独指出。
<table><tbody>
    <tr>
        <td align="center" valign="middle">选项</td>
        <td align="center" valign="middle">作用</td>
    </tr> 
    <tr>
        <td>-fsyntax-only</td>
        <td>检查程序中的语法错误,但是不产生输出信息</td>
    </tr>
    <tr>
        <td>-w</td>
        <td>禁止所有警告信息</td>
    </tr>    
    <tr>
        <td>-Wimplicit-int</td>
        <td>警告没有指定类型的声明</td>
    </tr>
    <tr>
        <td>-Wimplicit-function-declaration</td>
        <td>警告在声明之前就使用的函数</td>
    </tr>
    <tr>
        <td>-Wimplicit</td>
        <td>上面两个选项之和</td>
    </tr>
    <tr>
        <td>-Wreturn-type</td>
        <td>若函数定义了返回值，而无return语句，则警告</td>
    </tr>
    <tr>
        <td>-Wunused</td>
        <td>如果某个局部变量除了声明就没再使用,或者声明了静态函数但是没有定义,或者某条语句的运算结果显然没有使用, 编译器就发出警告</td>
    </tr>
    <tr>
        <td>-Wformat</td>
        <td>检查对 printf 和 scanf 等函数的调用,确认各个参数类型和格式串中的一致</td>
    </tr>
    <tr>
        <td>-Wuninitialized</td>
        <td>在初始化之前就使用自动变量.这些警告只可能做优化编译时出现,因为他们需要数据流信息,只有做优化的时候才估算数据流信息.如果不指定 `-O'选项,就不会出现这些警告</td>
    </tr>
    <tr>
        <td>-Wparentheses</td>
        <td>在某些情况下如果忽略了括号,编译器就发出警告</td>
    </tr>
    <tr>
        <th bgcolor=#9acd32>-Wall </td>
        <th bgcolor=#9acd32>结合所有上述的`-W'选项</td>
    </tr>
    <tr>
        <th bgcolor=#9acd32>-Werror</td>
        <th bgcolor=#9acd32>视警告为错误;出现任何警告即放弃编译</td>
    </tr>
</table> 












