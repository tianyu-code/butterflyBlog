---
title: makefile的参数
date: 2020-07-15 15:00:32
tags:
  - makefile
categories:
  - makefile
---


# 目录

+ 指定目标文件
+ make的参数

<!--------more------->


#  **指定目标文件**

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

#  **make的参数**







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












