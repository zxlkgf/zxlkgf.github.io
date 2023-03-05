# 端口复用


# 半关闭
```text
当TCP连接中A向B发送FIN请求关闭，另一端B回应ACK之后(A端进入FIN_WAIT_2状态)，并没有立即发送FIN给A，A方处于半连接状态(半开关),此时A可以接受B发送的数据，但是A已经不能再向B发送数据。  
```
从程序的角度可以使用API控制实现半连接状态:
```text
#include <sys/socket.h>
int shutdown(int sockfd,int how);
sockfd:需要关闭的sockfd描述符
how:允许为shutdown操作选择一下几种方式
    SHUT_RD(0):关闭sockfd上的读功能，此选项不允许sockfd进行读操作。该套接字不再接受数据，任何当前在套接字接受缓冲区的数据将被无声的丢弃掉。
    SHUT_WR(1):关闭sockfd的写功能，此选项不允许sockfd进行写操作，进程不能再对此套接字发出写操作。
    SHUT_PDWR(2):关闭sockfd的读写功能、
```
使用close终止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为0时才关闭连接。shutdown不考虑描述符的引用计数，直接关闭描述符。也可选择终止一个方向的连接。  

注意：  
1.如果有多个进程共享一个套接字，close没被调用一次，计数减1，直到计数为0时，也就是所有进程都调用了close，套接字被释放。  

2.在多进程中如果一个进程调用了shutdown(sfd,SHUT_RDWR)后，其它的进程将无法进行通信。但如果一个进程close(sfd)将不会影响进程。  

# 端口复用
端口复用的最常用的用途是：  
1.防止服务器重启时之前绑定的端口还未释放。  
2.程序突然退出而系统没有释放端口。  

```c++
#include <sys/types.h>
#include <sys/socket.h>
//设置套接字的属性
int setsockopt(int sockfd,int level,int optname,const void*optval,socklen_t optlen);
    参数
        -sockfd:要操作的文件描述符
        -level:级别 ：SOL_SOCKET（端口复用级别）
        -optname : 选项的名
            -SO_REUSEADDR
            -SO_REUSERPORT
        - optval: 端口复用的值(整型)
            -1:可以复用
            -0:不可以复用
        - optlen: optval参数的大小
端口复用，设置的时机是在服务器绑定端口之前、
```

查看网络相关信息的命令  
netstat:  
    参数:  
        -a 所有的socket  
        -p 显示正在使用socket的程序的名称  
        -n 直接使用IP地址，而不通过域名服务器  
        -l 显示正在监听的socket  
