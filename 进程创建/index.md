# 进程创建



# 进程的创建
## 01进程创建
系统允许一个进程创建新进程，新进程即为子进程，子进程还可以创建子进程，形成进程树结构模型  
#include <sys/types.h>  
#include <unistd.h>  
pid_t fork(void);  
返回值:  
    成功:子进程中返回0,父进程中返回子进程ID  
    失败:返回-1;  
    失败原因:  
        1.当前系统的进程数已经达到了系统规定的上限，这时errno的值被设置为EAGAIN  
        2.系统内存不足，这时errno的值为ENOMEM  

## fork()进程创建总结
首先父进程执行到fork的时候会创建子进程，fork后会给父子进程分别返回一个pid号（父进程fork后返回的pid是子进程的pid，子进程的pid为0），此时系统会将父进程的用户区数据和内核数据区拷贝过来生成一个虚拟地址空间供子进程使用，之后，如果没有执行写操作的时候，父子进程共同指向一个虚拟地址空间，但是一旦发生内存写操作，就会生成新的内存空间将父子进程的变量分开，防止内存碰撞。
