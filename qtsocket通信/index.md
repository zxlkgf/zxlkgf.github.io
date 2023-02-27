# QtSocket通信


# 

socket编程有三种

1.流式套接字(SOCKET_STREAM)

2.数据报式套接字(SOCKET_DGRAM)

3,原始套接字(SOCKET_RAW)

前两者较常使用，基于TCP使用的是SOCKET_STREAM(流式套接字)

服务端

1.socket():创建流式socket

```c++
int     socket(int, int, int);
头文件： 
#include <sys/types.h>
#include <sys/socket.h>
函数原型：
int socket(int domain, int type, int protocol)
    domain: 协议类型，一般为AF_INET
    type: socket类型(SOCKET_STREAM,SOCKET_DGRAM,SOCKET_RAW)
    protocol:用来指定socket所使用的传输协议编号，通常设为0即可
```

2.bind():指定用于通信的IP地址和port端口

```c++
bind()
头文件：
#include <sys/types.h>
#include <sys/socket.h>
函数原型：
int bind(int sockfd, struct sockaddr *my_addr, int addrlen)
    sockfd: socket描述符
    my_addr:是一个指向包含有本机ip地址和端口号等信息的sockaddr类型的指针
    addrlen:常被设为sizeof(struct sockaddr)
```

3.listen():把socket设为监听对象

```c++
3 listen()
头文件：
#include <sys/socket.h>
函数原型：
int listen(int sockfd, int backlog);
sockfd:socket()系统调用返回的socket描述符
backlog:指定在请求队列中的最大请求数，进入的连接请求将在队列中等待accept()它们。
```

4.accept():接受客户端发来的连接请求

```c++
4 accept()
头文件：   
#include <sys/types.h>
#inlcude <sys/socket.h>
函数原型：
int accept(int sockfd, void *addr, int addrlen)
    sockfd:是被监听的socket描述符
    addr:通常是一个指向sockaddr_in变量的指针，该变量用来存放提出连接请求服务的主机的信息
    addrlen:sizeof(struct sockaddr_in)
```

5.recv()/send():接受或者发送

```c++
send()
头文件：
#include <sys/socket.h>
函数原型：
int send(int sockfd, const void *msg, int len, int flags);
    sockfd:用来传输数据的socket描述符
    msg:要发送数据的指针 
    flags: 0

recv()
头文件：
#include <sys/types.h>
#include <sys/socket.h>
函数原型:
int recv(int sockfd, void *buf, int len, unsigned int flags)
   sockfd：接收数据的socket描述符
   buf:存放数据的缓冲区
   len:缓冲的长度
   flags:0
```

6.close():关闭socket连接

客户端

1.socket():创建流式套接字

2.connect():连接服务器，发起请求

3.send()/recv():接受或者发送

4.close():关闭socket连接 释放资源

## 备注

```c++
htons
头文件:
#include <arpa/inet.h>
uint16_t htons(uint16_t hostshort);　
htons的功能：
    将一个无符号短整型数值转换为网络字节序，即大端模式(big-endian)　
    参数u_short hostshort: 16位无符号整数　
返回值:
    TCP / IP网络字节顺序.


htonl()
#include <arpa/inet.h>　　
uint32_t htonl(uint32_t hostlong);　　
简述：将主机的无符号长整形数转换成网络字节顺序。　
hostlong：主机字节顺序表达的32位数。　　
注释：
  　　本函数将一个32位数从主机字节顺序转换成网络字节顺序。　　
返回值：　
     htonl()返回一个网络字节顺序的值。
```

```c++
使用gethostname(const char *name);
返回如下结构体
struct hostent {
      char  *h_name;            /* official name of host */
      char **h_aliases;         /* alias list */
      int    h_addrtype;        /* host address type */
      int    h_length;          /* length of address */
      char **h_addr_list;       /* list of addresses */
}
hostent->h_name
表示的是主机的规范名。例如www.baidu.com的规范名其实是www.a.shifen.com。

hostent->h_aliases
表示的是主机的别名。www.baidu.com就是baidu他自己的别名。

hostent->h_addrtype    
表示的是主机ip地址的类型。只会是ipv4(AF_INET)， 这个函数处理不了ipv6

hostent->h_length      
表示的是主机ip地址的长度。

hostent->h_addr_list
表示的是主机的ip地址。是网络字节序，需要通过inet_ntop函数转换。
```

