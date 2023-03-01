# 信号


# 信号的概念
1.信号是Linux进程间通信的最古老的方式之一，是事件发生时对进程的通知机制，有时也称之为软件中断，它是在软件层次上对中断机制的一种模拟，是一种异步通信的方式，信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某个突发事件  

2.发往进程的诸多信号，通常都源于内核。引发内核为进程产生信号的各类事件如下：  
2.1 对于前台进程，用户可以通过输入特殊的终端字符给它发送信号。比如输入Ctrl+C通常会给进程发送一个终端信号  

2.2 硬件发生异常，及硬件检测到一个错误条件并通知内核，随机再由内核发送响应信号给相关进程。比如执行一条异常的机器语言指令，比如被0除，或者引用了无法访问的内存区域  

2.3 系统状态变化，比如alarm定时器到期引起SIGALRM信号，进程执行cpu时间超限，或者该进程的某个子进程退出  

2.4 运行kill命令 或者 kill函数  

---

## 使用信号的目的   
1.使用信号的两个主要目的:  
    1.1让进程知道已经发生了特定的事情  
    1.2强迫进程执行他自己代码中的信号处理程序  
2.信号的特点  
    2.1 简单  
    2.2 不能携带大量信息  
    2.3 满足某个特定条件才发送  
    2.4 优先级比较高  
3.查看系统定义的信号列表 kill -i  
4.前31个信号为常规信号，其余为实时信号  

---
## 信号列表

|编号|信号名称|对应事件|默认动作|
| ------ | ------ | ------ | ------ |
|1|SIGHUP|用户退出shell时，由该shell启动的所有进程将收到这个信号|终止进程|
|2|SIGINT|当用户按下CTRL+C组合键时，用户终端向正在运行中的由该终端启动的程序发送此信号|终止进程|
|3|SICQUIT|用户按下CTRL+\组合键时产生该信号，用户终端向正在运行中的由该终端启动的程序发送该信号|终止进程|
|4|SIGILL|CPU检测到某进程执行了非法指令|终止并产生core文件|
|5|SIGTRAP|该信号由断点指令或其他trap指令产生|终止并产生core文件|
|6|SIGABRT|调用abort函数时产生该信号|终止并产生core文件|
|7|SIGBUS|非法访问内存地址，包括内存对其出错|终止并产生core文件|
|8|SIGFPE|在发生致命的运算错误时发出，不仅包括浮点运算错误，还包括溢出及除数为0等所有的算法错误|终止并产生core文件|
|9|SIGKILL|无条件终止进程，该信号不能被忽略，处理和阻塞|终止进程，可以杀死任何进程|
|10|SIGUSEL|用户定义的信号，即程序员可以再程序中定义并使用|终止进程|
|11|SIGSECV|指示进程进行了无效内存访问(段错误)|终止并产生core文件|
|12|SIGUSR2|另外一个用户自定义信号，程序员可以在程序中定义并使用|终止进程|
|13|SIGPIPE|Broken pipe向一个没有读端的管道写数据|终止进程|
|14|SIGALRM|定时器超时，超时的时间，由系统调用alarm设置|终止进程|
|15|SIGTERM|程序结束信号，与SIGKILL不同的是，该信号可以被阻塞和终止。通常用来表示程序正常退出。执行shell命令kill时，缺省产生这个信号|进程终止|
|16|SIGSTKFLT|LINUX早期版本出现的信号，现在仍然保留|程序终止|
|17|SIGCHLD|子进程结束时，父进程会收到这个信号|忽略信号|
|18|SIGCONT|如果进程已停止，则使其继续运行|继续/忽略|
|19|SIGSTOP|停止进程的执行，信号不能被忽略，处理和阻塞|为终止进程|
|20|SIGTSTP|停止终端交互进程的运行，按下CTRL+X发出该信号|暂停进程|
|21|SIGTTIN|后台程序读终端控制台|暂停进程|
|22|SIGTTOU|该信号类似于SIGTTIN，后台进程向终端发送数据|暂停进程|
|23|SIGURG|套接字上有紧急数据时，向当前正在运行的进程发出信号，报告有紧急数据到达，如网络带外数据到达|忽略该信号|
|24|SIGXCPU|进程执行时间超过了分配给该进程的CPU时间，系统产生该信号并发送给进程|终止进程|
|25|SIGXFSZ|超过文件的最大长度设置|终止进程|
|26|SIGVTALRM|虚拟时钟超时时产生该信号，类似于SIGALRM，但是该信号只计算该进程占用CPU的使用时间|终止进程|
|27|SIGPROF|类似于SIGVTALRM，它不仅包括该进程占用CPU时间还包括执行系统调用时间|终止进程|
|28|SIGWINCH|窗口大小变化时发出|忽略|
|29|SIGIO|此信号向进程指示发出一个异步IO事件|忽略|
|30|SIGPWR|关机|终止进程|
|31|SIGSYS|无效系统调用|终止进程并产生core文件|
|34-64|SIGRTIMN—SIGRTMAX|没有固定含义|终止进程|
---
## 信号的5种默认处理动作
1.查看信号的详细信息: man 7 signal  
2.信号的5种默认处理动作  
2.1 Term 终止进程  
2.2 Ign  当前进程忽略掉这个信号  
2.3 Core 终止进程，并产生一个Core文件  
2.4 Stop 暂停当前进程  
2.5 Cont 继续执行当前被暂停的进程  
信号的几种状态：产生，未决，递达  
SIGKILL 和 SIGSTOP信号不能被捕捉，阻塞或忽略

### 信号相关函数
```c++
int kill(pid_t pid,int sig);
int raise(int sig);
void abort(void);
unsigned int alarm(unsigned int seconds);
in setitimer(int which,const struct itimerval *new_val,struct itimerval * old_value);
```
kill,raise,abort函数详细
```text
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid,int sig);
    作用:给某个进程pid，发送信号sig
    参数:
        pid:需要发送的进程的pid
        pid>0:将信号发送给指定的进程
        pid=0：将信号发送给当前的进程组
        pid=-1:将信号发送给每一个有权限接受这个信号的进程
        pid<-1：pid=某个进程组的id取反，给进程组内所有成员发送信号
        sig:需要发送的sig编号或者宏值
    例如：kill(getppid(),9);

int raise(int sig);
    功能:给当前进程发送信号
    参数:
        sig:需要发送的sig编号或者宏值
    返回值：
        成功0
        失败除0外的数字
void abort(void);
    功能：发送SIGABRT信号给当前进程，杀死当前进程
kill(getpid(),SIGABRT);
```
alarm(),setitimer(..)

```text
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
    功能:定时
    参数：seconds 秒
    返回值：返回调用时的时间


#include <sys/time.h>
int setitimer(int which, const struct itimerval *new_value,
              struct itimerval *old_value);
    功能:设置定时器(闹钟)，可以替代alarm函数，精度好，实现周期性定时
    参数:
        which：定时器以什么时间计时
            ITIMER_REAL:真实时间,时间到达，发送SIGALRM  常用的
            ITIMER_VIRTUAL:用户时间，时间到达，发送SIGVTALRM
            ITIMER_PROF:以该进程在用户态和内核态下所消耗额时间来计算，时间到达，发送SIGPROF
        
        new_value:设置定时器的属性,
            struct itimerval {          //定时器的结构体
               struct timeval it_interval; //间隔时间
               struct timeval it_value;    //延迟多长时间执行定时器
           };
            struct timeval {            //时间的结构体
               time_t      tv_sec;        //  秒数
               suseconds_t tv_usec;       // 微秒
           };

        old_value:记录上一次的定时的时间参数，一般不使用，可以指定NULL

    返回值:
        成功：0
        失败:-1  设置errno

```

---

### 信号捕捉函数
```c++
sighandler_ signal(int signum,sighandler_t handler);
int sigaction(int signum,const struct sigaction *act,struct sigaciton *oldact);
```
sighandler_ signal(int signum,sighandler_t handler);
```text
/*
    #include <signal.h>

    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);
    功能:

        参数:
            -signum:要捕捉的信号
            -handler:捕捉到信号如何处理
                SIG_IGN:忽略，捕捉到就忽略 进程不终止
                SIG_DFL:使用信号默认的行为，原本的行为
                回调函数:去执行操作
        返回值:
            成功:返回上一次注册的信号处理函数的地址，第一次调用返回NULL
            失败:返回SIG_ERR，设置errno
        回调函数:
            -需要程序员自己实现，提前准备好，函数的类型根据实际需要
            -不是程序员调用，而是当信号产生的时候，内核自动调用
*/

```
int sigaction(int signum,const struct sigaction *act,struct sigaciton *oldact);
```text
    #include <signal.h>
    int sigaction(int signum, const struct sigaction *act,
                            struct sigaction *oldact);

        - 功能：检查或者改变信号的处理。信号捕捉
        - 参数：
            - signum : 需要捕捉的信号的编号或者宏值（信号的名称）
            - act ：捕捉到信号之后的处理动作
            - oldact : 上一次对信号捕捉相关的设置，一般不使用，传递NULL
        - 返回值：
            成功 0
            失败 -1

     struct sigaction {
        // 函数指针，指向的函数就是信号捕捉到之后的处理函数
        void     (*sa_handler)(int);
        // 不常用
        void     (*sa_sigaction)(int, siginfo_t *, void *);
        // 临时阻塞信号集，在信号捕捉函数执行过程中，临时阻塞某些信号。
        sigset_t   sa_mask;
        // 使用哪一个信号处理对捕捉到的信号进行处理
        // 这个值可以是0，表示使用sa_handler,也可以是SA_SIGINFO表示使用sa_sigaction
        int        sa_flags;
        // 被废弃掉了
        void     (*sa_restorer)(void);
    };

```
---
### 信号集
1.许多信号相关的系统调用都需要能够表示一组不同信号，多个信号可使用一个称之为信号集的数据结构表示，其系统数据类型为sigset_t。  

2.在PCB中有两个非常重要的信号集，一个称之为'阻塞信号集'，另一个称之为'未决信号集'。这两个信号集都是内核是由位图机制来实现的。但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对PCB中的这两个信号集进行修改。  

3.信号的'未决'是一种状态，指的是从信号的产生到信号被处理前的这一段时间。  

4.信号的'阻塞'是一个开关动作,指的是阻止信号被处理，而不是阻止信号产生。  

5.信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感操作。

```text
    以下信号集相关的函数都是对自定义的信号集进行操作。

    int sigemptyset(sigset_t *set);
        - 功能：清空信号集中的数据,将信号集中的所有的标志位置为0
        - 参数：set,传出参数，需要操作的信号集
        - 返回值：成功返回0， 失败返回-1

    int sigfillset(sigset_t *set);
        - 功能：将信号集中的所有的标志位置为1
        - 参数：set,传出参数，需要操作的信号集
        - 返回值：成功返回0， 失败返回-1

    int sigaddset(sigset_t *set, int signum);
        - 功能：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号
        - 参数：
            - set：传出参数，需要操作的信号集
            - signum：需要设置阻塞的那个信号
        - 返回值：成功返回0， 失败返回-1

    int sigdelset(sigset_t *set, int signum);
        - 功能：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号
        - 参数：
            - set：传出参数，需要操作的信号集
            - signum：需要设置不阻塞的那个信号
        - 返回值：成功返回0， 失败返回-1

    int sigismember(const sigset_t *set, int signum);
        - 功能：判断某个信号是否阻塞
        - 参数：
            - set：需要操作的信号集
            - signum：需要判断的那个信号
        - 返回值：
            1 ： signum被阻塞
            0 ： signum不阻塞
            -1 ： 失败
```
---

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
int sigpending(sigset_t *set);
```text
    int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
        - 功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
        - 参数：
            - how : 如何对内核阻塞信号集进行处理
                SIG_BLOCK: 将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变
                    假设内核中默认的阻塞信号集是mask， mask | set
                SIG_UNBLOCK: 根据用户设置的数据，对内核中的数据进行解除阻塞
                    mask &= ~set
                SIG_SETMASK:覆盖内核中原来的值
            
            - set ：已经初始化好的用户自定义的信号集
            - oldset : 保存设置之前的内核中的阻塞信号集的状态，可以是 NULL
        - 返回值：
            成功：0
            失败：-1
                设置错误号：EFAULT、EINVAL

    int sigpending(sigset_t *set);
        - 功能：获取内核中的未决信号集
        - 参数：set,传出参数，保存的是内核中的未决信号集中的信息。
```

### SIGCHLD信号
1.SIGCHLD信号产生的条件  
1.1 子进程终止时  
1.2 子进程接收到SIGSTOP信号停止时  
1.3 子进程处在停止态，接受SIGCONT后唤醒时  
2 以上三种条件都会给父进程发送SIGCHLD信号，父进程默认忽略该信号  

