---
title: GDB调试之改变程序执行流程
date: 2020-04-06 00:02:35
tags:
  - GDB
categories:
  - GDB
keywords: GDB, 调试, 改变程序执行流程
description: 本文介绍一次使用GDB改变程序执行流程的实验过程

---


# 目录

+ 目的
+ 实例


#  **目的**

最近学习了函数调用及返回的流程后，突发奇想能不能直接使用GDB修改栈空间，达到`调用函数 -> 返回主函数`
变为  `调用函数 -> 插入自定义函数 -> 返回主函数`的效果，感觉很有意思，遂进行以下实验

<!--------more------->

#  **实例**

肥肠简单的代码如下

```
int add2(int a, int b)
{
  printf("asdasdads\n");
  return 1;
}


int add(int a, int b)
{

   return a+b;
}

int main(int argc)
{
    int answer;
    getchar();
    answer = add(40, 2);
    printf("after add\n");
}
```
程序正常执行的效果如下
```
[root@VM_0_4_centos stackBreak]# ./main

after add
[root@VM_0_4_centos stackBreak]#
```

## 理论分析

接下来进行我们的实验，首先放上理论的修改方法

1. 程序执行到进入`add`函数时，栈的**示意图**如下

<image src=/images/改写程序执行流程/程序栈示意图.png width="70%">

2. add函数执行完毕返回时，需要依次弹栈给`rbp`和`rip`两个寄存器，若正常执行程序，rip寄存器中将保存add函数调用后的地址，然后即可正常返回，我们此处改写该位置的栈内容，将add2函数的指令地址写入，如图所示

<image src=/images/改写程序执行流程/第一次破坏返回地址.png width="70%">

3. 到此时应该就能跳转到我们指定的函数中了，但为了能够安全返回，还需要将add2函数执行完毕后，弹栈到`rip`寄存器的指令地址改写，如下图所示

<image src=/images/改写程序执行流程/第二次破坏返回地址.png width="70%">


## 使用GDB实战

**理论分析完毕后接下来就是实战啦**，为了程序执行的效果更佳明显，采用attach的方式调试
1. 在调用`add`函数之前打断点

<image src=/images/改写程序执行流程/第一次断点.png width="70%">

2. 使用`si`单步运行汇编代码，查看栈内容

<image src=/images/改写程序执行流程/进入add函数.png width="70%">

3. 此时我们改写此时的栈内容，将其改写成`add2`函数的指令地址，这样在`add`函数返回时，就会跳转到`add2`函数啦

<image src=/images/改写程序执行流程/第一次改写栈内容.png width="70%">

4. 此时我们再次改写栈内容为调用`add`函数后返回`main`的指令地址

<image src=/images/改写程序执行流程/第二次改写栈内容.png width="70%">

5. 函数`add2`执行完毕后，安全返回

<image src=/images/改写程序执行流程/完成跳转.png width="70%">

6. GDB直接continue执行，发现程序无报错

<image src=/images/改写程序执行流程/实验完成.png width="70%">


## **总结：**

至此我们的实验就大功告成了，达到了`调用函数 -> 返回主函数`变为  `调用函数 -> 插入自定义函数 -> 返回主函数`的效果。
通过本次学习，感觉对自己的程序真是可以`为所欲为 -> 为所欲为 -> 为所欲为`

<image src=/images/为所欲为.jpg width="20%">
