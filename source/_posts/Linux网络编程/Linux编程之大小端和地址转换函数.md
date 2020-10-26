---
title: Linux编程之大小端和地址转换函数
date: 2020-08-19 14:00:32
tags:
  - 网络编程
  - 大小端和地址转换
categories:
  - Linux网络编程

keywords: Linux,网络编程,大小端和地址转换
cover: /images/封面图/MAC.jfif
top_img: /images/封面图/MAC.jfif
description: 本文介绍Linux下编程的大小端转换函数以及地址转换函数。
sticky: 0

---

#  地址结构介绍

## IPV4地址结构

```C
struct  in_addr {
	in_addr_t      s_addr;
}

struct sockaddr_in {
  sa_family_t   sin_family;	/*2byte AF_INET */
  in_port_t     sin_port;	/* 2byte Port number*/
  struct in_addr sin_addr;	/* 4byte Internet address*/
  char          sin_zero[8];/*8byte unused*/
}
```

1. IPV4地址结构至少16字节
2. sin_port端口号和sin_addr_t地址均以网络字节序存储
3. struct in_addr结构体的历史：为什么这个结构体就一个成员还使用结构体呢？早期版本将`struct in_addr`定义为多种结构的联合（union），方便访问32位地址中的每个字节，这在使用ABC类地址时非常方便，随着子网划分技术和无类地址编排的出现，这个联合已经不需要了，所以struct in_addr中仅剩下了一个成员。

## 通用套接字地址

socket各种函数传参时，需要兼容各类地址结构，所以使用通用套接字地址进行类型转换
```C
struct sockaddr {
	sa_family_t	sa_family;	/* address family, AF_xxx	*/
	char		sa_data[14];/* 14 bytes of protocol address	*/
}
```

# 大小端转换函数

x86架构都是小端模式，而网络协议是大端模式,并且Linux下网络编程的系统调用中也有使用网络字节序的参数，所以大小端转换非常有必要

<table>
    <thead>
      <tr>
          <th align="center" valign="middle" style="width: 300px;">功能</th>
          <th align="center" valign="middle" >函数</th>
      </tr>
    </thead>
    <tbody>
       <tr>
          <td align="center" valign="middle">主机字节序转换成网络字节序</td>
          <td>uint32_t htonl(uint32_t hostlong);<br>
              uint16_t htons(uint16_t hostshort);</td>
       </tr>     
      <tr>
          <td align="center" valign="middle">网络字节序转换成主机字节序</td>
          <td>uint32_t ntohl(uint32_t netlong);<br>
                uint16_t ntohs(uint16_t netshort);
</td>
      </tr>  
    </tbody>
</table>


# 地址转换函数

## inet_aton

```C
int inet_aton(const char *cp, struct in_addr *inp);
功能：将cp指向的点分十进制字符串转换为网络字节序的32位地址
成功返回1，失败返回0
```

## inet_ntoa

```C
char *inet_ntoa(struct in_addr in);
功能：将网络字节序的32位地址转换为点分十进制字符串，通过返回值返回
```

## inet_addr

```C
 in_addr_t inet_addr(const char *cp);
功能：将cp指向的点分十进制字符串转换为网络字节序的32位地址,通过返回值返回
注意：该函数出错时返回INADDR_NONE,该宏32位均位1，这意味着255.255.255.255不能由该函数转换，尽量不要使用该函数
```

## inet_pton
```C
int inet_pton(int family,const char *strptr, void *addrptr);
功能：将strptr指向的点分十进制数串转换成网络字节序的32位地址，
family指定协议：AF_INET或AF_INET6
成功返回1，失败返回0
```

## inet_ntop
```C

const char *inet_ntop(int family,const void *addrptr,char *strptr, size_t len);
功能：将addrptr指向的网络字节序的32位地址转换成点分十进制数串
len指出了strptr的空间大小，其取值如下
#define INET_ADDRSTRLEN 16 //for ipv4
#define INET6_ADDRSTRLEN 46 //for ipv6
因为ip地址的字符串最多有15位，最后还有一个'\0'，所以注意使用这个函数的时候空间要+1
成功返回strptr的地址，失败返回NULL

```
---