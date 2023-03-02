# 线程


## 线程概述
1.与进程(process)类似，线程(thread)是允许应用程序并发执行多个任务的一种机制。一个进程可以包含多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份内存区域，其中包括初始化数据段，未初始化数据段，以及堆内存段。(传统意义上的UNIX进程只是多线程程序的一个特例，该进程只包含一个线程)。  

2.进程是CPU分配资源的最小单位，线程是操作系统调度执行的最小单位。  

3.线程是轻量级的进程(LWP:Light Weight Process)，在Linux环境下线程的本质是进程。  

4.查看指定进程的LWP号:ps -Lf pid  

### 线程和进程的区别
1.进程间的信息难以共享，由于除去只读代码之外，父子进程并未共享内存，因此必须采用一些进程间通信的方式，在进程间进行信息交换。  

2.调用fork()来创建进程的代价相对较高，即便利用写时复制技术，但仍然需要复制诸如内存页表和文件描述符表之类的多种进程属性，这意味着fork()调用在时间上的开销依然不菲。  

3.线程之间能够方便，快速的共享信息。只需要将数据复制到共享(全局或堆)变量中即可。  

4.创建线程比创建进程通常要快10倍甚至更多。线程间是共享虚拟地址空间的，无需采用写时复制来复制内存，也无需复制页表。  

### 线程之间共享和非共享资源
|共享资源|
|--------|
|进程ID和父进程ID|
|进程组ID和会话ID|
|用户ID和用户组ID|
|文件描述符|
|信号处理|
|文件系统的相关信息:文件权限掩码，当前工作目录|
|虚拟地址空间(除栈,text)

---
|非共享资源|
|--------|
|线程ID|
|信号掩码|
|线程特有数据|
|error变量|
|实时调度策略和优先级|
|栈，本地变量和函数的调用链接信息|

---

## 线程的创建
线程操作函数
```c++
pthread_t pthread_self(void);
int pthread_equal(pthread_t t1,pthread_t t2);
int pthread_create(pthread_t *thread,const pthread_attr_t *attr,void *(*start_routine)(void *),void *arg);
void pthread_exit(void *retval);
int pthread_join(pthread_t thread,void **retval);
int pthread_detach(pthread_t thread);
int pthread_cancel(pthread_t thread);
```
---
int pthread_create(pthread_t *thread,const pthread_attr_t *attr,void *(*start_routine)(void *),void *arg);

```C++
/*

    一般情况下，main函数所在的线程称之为主线程，之后创建的称之为子线程
    #include <pthread.h>
    int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);
    作用:创建一个子线程
    参数:
        pthread_t *thread:传出参数，代表的是线程创建成功之后，子线程的线程ID会写到这个参数里
        const pthread_attr_t *attr:需要设置的线程的属性，一般使用默认值，NULL
        void *(*start_routine) (void *):函数指针，子线程需要处理的逻辑代码
        void *arg:给第三个参数使用，传参
    返回值：
        成功:0
        错误:失败会返回错误号码，这个错误号和之前的errno不太一样
        不能通过perror()，而是使用char *strerror(int errornum);
*/
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

//子线程需要处理的逻辑代码
void *callback(){
    printf("chil thread。。。。\n");
    return NULL;
}

int main(){
    //创建一个线程
    pthread_t tid;
    int ret = pthread_create(&tid,NULL,callback(),NULL);

    if(ret !=0){
        char *str = strerror(ret);
        printf("error : %s\n",str);
    }

    for(int i = 0;i < 5;i++){
        printf("%d\n",i);
    }
    sleep(1);


    return 0;
}
```
---
void pthread_exit(void *retval);

```c++
/*
    #include <pthread.h>
    void pthread_exit(void *retval);
    作用L:终止一个线程，在哪个线程调用，就终止哪个线程
    参数:
        -retval:需要传递一个指针，作为一个返回值，可以再pthread_join()中获取到

    pthread_t pthread_self(void);
    作用:获取当前线程的ID

*/
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>

void *callback(){
    printf("child thread id:%ld\n",pthread_self());
    return NULL;
}

int main(){
    //创建一个子线程
    pthread_t tid;
    int ret = pthread_create(&tid,NULL,callback,NULL);
    if(ret!=0){
        char *str = strerror(ret);
        printf("%s\n",str);
        exit(0);
    }

    //主线程
    for(int i = 0;i < 5;i++){
        printf("%d\n",i);
    }

    printf("tid : %ld ,main thread id:%ld\n",tid,pthread_self());
    
    //让主线程退出，当主线程退出时，不会影响其他正常运行的线程
    pthread_exit(NULL);

    return 0;
}

```
---
int pthread_join(pthread_t thread,void **retval);
```c++
/*
    #include <pthread.h>
    int pthread_join(pthread_t thread, void **retval);
    作用：和一个终止的线程进行连接
        回收子线程的资源
        这个函数是阻塞函数，调用一次只能回收一个子线程
        一般在主线程中去使用
    参数:
        -thread：需要回收的子线程的id
        -retval: 接受子线程退出时的返回值
    返回值：
        成功返回0
        失败返回错误号
*/
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>

void *callback(){
    printf("child thread id:%ld\n",pthread_self());
    int num = 10;
    pthread_exit((void *)&num);
}

int main(){
    //创建一个子线程
    pthread_t tid;
    int ret = pthread_create(&tid,NULL,callback,NULL);
    if(ret!=0){
        char *str = strerror(ret);
        printf("%s\n",str);
        exit(0);
    }

    //主线程
    for(int i = 0;i < 5;i++){
        printf("%d\n",i);
    }

    printf("tid : %ld ,main thread id:%ld\n",tid,pthread_self());

    //主线程调用pthread_join去回收子线程的资源
    int  *thread_return_value = NULL;

    ret  =  pthread_join(tid,(void **)&thread_return_value);
    if(ret!=0){
        char *str = strerror(ret);
        printf("%s\n",str);
        exit(0);
    }
    printf("exit date : %d\n",*thread_return_value);

    printf("回收子线程资源成功\n");
    
    //让主线程退出，当主线程退出时，不会影响其他正常运行的线程
    pthread_exit(NULL);

    return 0;
}
```
---
int pthread_detach(pthread_t thread);
```c++
/*

    #include <pthread.h>
    int pthread_detach(pthread_t thread);
    作用:分离一个线程
    参数:
        -pthread_t thread:传入一个线程ID,将指定ID的线程标记为分离
    返回值：
        成功返回 0
        错误返回 error number

    注意：
    1.不可多次分离，否则会产生不可预料的行为。
    2.不可以join一个已经detach的线程
*/

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

void * callback(void * arg){
    printf("child thread id : %ld\n",pthread_self());
    return NULL;
}

int main(){
    //创建一个线程
    pthread_t tid;

    int ret = pthread_create(&tid,NULL,callback,NULL);
    if(ret != 0){
        char * str = strerror(ret);
        printf("error1 :%s\n",str);
        exit(0);
    }

    // 输出主线程和子线程的id
    printf("tid = %ld,main thread id = %ld\n",tid,pthread_self());

    //设置子线程分离,子线程结束时对应的资源就不需要主线程释放
    ret = pthread_detach(tid);
    if(ret != 0){
        char * str = strerror(ret);
        printf("error2 :%s\n",str);
        exit(0);
    }
    //设置分离后，对分离的子线程进程链接，pthread_join()//Invalid argument
    ret = pthread_join(tid,NULL);
    if(ret != 0){
        char * str = strerror(ret);
        printf("error3 :%s\n",str);
        exit(0);
    }

    pthread_exit(NULL);

    return 0;
}
```
---
int pthread_cancel(pthread_t thread);

```c++
/*

    #include <pthread.h>
    int pthread_cancel(pthread_t thread);
    作用:取消线程(让线程终止)
        取消某个线程，可以终止某个线程的运行
        但是并不是立马终止，而是当子线程执行到一个取消点，线程才会终止
        取消点:系统规定好的一些系统调用，我们可以粗略的理解为从用户区切换到内核区
    参数:
        -pthread_t thread:需要取消的线程ID
    返回值:
        成功返回0，失败返回非0 错误号码
*/

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

void * callback(void * arg){
    printf("child thread id : %ld\n",pthread_self());
    for(int i=0;i<20;i++){
        printf("child i = %d\n",i);
    }
    return NULL;
}

int main(){
    //创建一个线程
    pthread_t tid;

    int ret = pthread_create(&tid,NULL,callback,NULL);
    if(ret != 0){
        char * str = strerror(ret);
        printf("error1 :%s\n",str);
        exit(0);
    }
    //取消线程
    pthread_cancel(tid);

    for(int i=0;i<5;i++){
        printf("parent i = %d\n",i);
    }
    // 输出主线程和子线程的id
    printf("tid = %ld,main thread id = %ld\n",tid,pthread_self());

    pthread_exit(NULL);

    return 0;
}
```
---

### 线程属性操作函数
```c++
int pthread_attr_init(pthread_addr_t *attr);
    -初始化线程属性变量
int pthread_attr_destory(pthread_addr_t *addr);
    -释放线程资源
int pthread_attr_getdetachstate(const pthread_attr_t *attr,int *detachstate);
    -获取线程分离的状态属性
int pthread_attr_setdetachstate(pthread_attr_t *addr,int detachstate);
    -设置线程分离的状态属性

```

在终端输入 man pthread_attr  + Tab 可以查看其余函数  


