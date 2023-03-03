# 线程同步+锁


## 线程同步
1.线程的主要优势在于，能够通过全局变量来共享信息。不过，这种便捷的共享是有代价的，必须确保多个线程不会同时修改同一个变量，或者某个线程不会读取正在由其他线程修改的变量。  

2.临界区是指访问某一种共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应该终端该片段的执行，  

3.线程同步：即当有一个线程在对内存进程操作时，其他线程不可以对这个内存地址进行操作，知道该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。  


### 互斥锁
1.为了避免线程更新共享变量时出现问题，可以使用互斥量(mutex是mutual exclusion的缩写)来确保同时仅用一个线程可以访问某项共享资源。可以使用互斥量来保证对任意共享资源的原子访问。  

2.互斥量有两种状态:已锁定(locked)和未锁定(unlocked),任何时候，至多只有一个线程可以锁定该互斥量。试图对已经锁定的某一互斥量再次加锁，将可能阻塞线程或者报错失败，具体取决于加锁时使用的方法。  

3.一旦线程锁定互斥量，随即成文该互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每一个共享资源(可能由多个相关变量组成)会使用不同的互斥量。每一个线程在访问同一资源时将采用如下协议:  
3.1 针对共享资源锁定互斥量  
3.2 访问共享资源   
3.3 对互斥量解锁  

### 互斥量
如果多个线程试图执行这一块代码(一个临界区),事实上只有一个线程能够持有该互斥量(其他线程将遭到阻塞),即同时只有一个线程能够进入这段代码区域，如下图所示。  
!(mutex)(../images/thread/1.png)

```c++
//互斥量的类型 pthread_mutex_t
int pthread_mutex_init(pthread_mutex_t *restrict mutex,const pthread_mutex_t *restrict mutex);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthraed_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
示例代码  
```c++
/*
//互斥量的类型 pthread_mutex_t
int pthread_mutex_init(pthread_mutex_t *restrict mutex,const pthread_mutex_t *restrict attr);
    -初始化互斥量
    -参数
        -mutex:需要初始化的互斥量变量
        -attr:互斥量相关的属性，NULL 或者自己写入
    - restrict：c语言修饰符，被修饰的指针，不能由另外的一个指针进行操作
int pthread_mutex_destroy(pthread_mutex_t *mutex);
    -释放互斥量
int pthread_mutex_lock(pthread_mutex_t *mutex);
    -加锁 阻塞的    如果有一个线程加锁了，其他的线程只能阻塞等待
int pthread_mutex_trylock(pthraed_mutex_t *mutex);
    -尝试加锁
int pthread_mutex_unlock(pthread_mutex_t *mutex);
    -解锁
*/

#include <pthread.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>



int ticket = 1000;
//创建一个互斥量
pthread_mutex_t mutex;

void * callback(void * arg){
    //卖票
    while(1){
        pthread_mutex_lock(&mutex);
        if(ticket>0){
            printf("thread id = %ld,正在卖第%d张票\n",pthread_self(),ticket);
            ticket--;
        }else{
            pthread_mutex_unlock(&mutex);
            break;
        }
        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}

int main(){
    //初始化互斥量
    pthread_mutex_init(&mutex,NULL);
    //创建一个线程
    pthread_t tid1;
    pthread_t tid2;
    pthread_t tid3;

    //创建
    pthread_create(&tid1,NULL,callback,NULL);
    pthread_create(&tid2,NULL,callback,NULL);
    pthread_create(&tid3,NULL,callback,NULL);

    //回收
    pthread_join(tid1,NULL);
    pthread_join(tid2,NULL);
    pthread_join(tid3,NULL);
    // //分离
    // pthread_detach(tid1);
    // pthread_detach(tid2);
    // pthread_detach(tid3);

    //主线程退出
    pthread_exit(NULL);

    //释放互斥量
    pthread_mutex_destroy(&mutex);

    return 0;
}
```
---

### 死锁
1.有时候，一个线程需要用时访问两个或者更多不同的共享资源，而每个资源又都由不同的互斥量管理。当超过一个线程加锁同一组互斥量时，就有可能发生死锁。  

2.两个或两个以上的进程在执行过程中，因争夺共享资源而造成一种互相等待的线程，若无外力作用，它们都无法推进下去。此时称系统处于死锁状态或者处于死锁。  

3.死锁的集中场景：  
3.1 忘记释放锁  
3.2 重复加锁  
3.3 多线程多锁，抢占锁资源  

---

### 读写锁
1.当有一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读取这个共享资源，但是由于互斥锁的排他性，所有其它线程都无法访问数据锁，也就是无法访问共享资源，但是实际上多个线程同时访问共享资源也不会导致问题。  

2.在对数据的读写操作时，更多的是读操作，写操作较少，例如对数据库数据的读写和应用。为了满足当前能够允许多个读出，但只允许一个写入的需求，线程提供了读写锁来实现。  

3.读写锁的特点:  
3.1 如果有其它线程读数据，则允许其它线程执行读操作，但是不允许写操作。  

3.2 如果有其它线程写数据，则其他线程不允许读，写操作。  

3.3 写是独占，写的优先级高  

#### 读写锁相关操作函数
```c++
//读写锁的类型 pthread_rwlock_t
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,const pthread_rwlockattr_t *restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
int phtread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trydlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

---

## 生产者消费者模型
生产者消费者模型，即多个生产者生产，多个消费者消费，共同操作一个变量，如果不加锁，则会导致内存越界，报错。但是加了锁，由于没有提醒机制，某个线程会单方面的阻塞，也不利于流程的实施。
```c++
/*
生产者消费者模型
*/

#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

struct Node
{
    int num;
    struct Node * next;
};
//头结点
struct Node * head = NULL;
//定义互斥锁
pthread_mutex_t mutex;

void * producer(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        struct Node * newNode = (struct Node *)malloc(sizeof(struct Node));
        newNode->next = head;
        head = newNode;
        newNode->num = rand()%1000;
        printf("add node,num : %d,tid : %ld \n",newNode->num,pthread_self());
        pthread_mutex_unlock(&mutex);
        usleep(100);
    }
    return NULL;
}

void * customer(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        struct Node * temp = head;
        if(head!=NULL){
            head = head->next;
            printf("del node num : %d, tid : %ld\n",temp->num,pthread_self());
            temp->next = NULL;
            free(temp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        }else{
            pthread_mutex_unlock(&mutex);
        }
    }
    return NULL;
}


int main(){
    //创建互斥锁
    pthread_mutex_init(&mutex,NULL);
    //创建5个生产，5个消费
    pthread_t ptids[5];
    pthread_t ctids[5];
    for(int i=0;i<5;i++){
        pthread_create(&ptids[i],NULL,producer,NULL);
    }

    for(int i=0;i<5;i++){
        pthread_create(&ctids[i],NULL,customer,NULL);
    }
    //线程连接
    for(int i=0;i<5;i++){
        pthread_join(ptids[i],NULL);
        pthread_join(ctids[i],NULL);
    }
    // while(1){
    //     sleep(10);
    // }
    //
    //消除互斥锁
    pthread_mutex_destroy(&mutex);
    pthread_exit(NULL);

    return 0;
}
```

---
##### 条件变量
使用条件变量可以根据线程内的运行，适当的阻塞线程 或者唤醒线程
```c++
//条件变量的类型 pthread_cond_t
int pthread_cond_init(pthread_cond_t *restrict cond,const pthread_condattr_t *restrict attr);
int pthread_cond_destroy(pthread_cond_t *cond);
int pthread_cond_wait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex);
int pthread_cond_timedwait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex,const struct timespec *restrict abstime);
int pathread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthead_cond_t *cond);
```
代码示例
```c++
/*
//条件变量的类型 pthread_cond_t
int pthread_cond_init(pthread_cond_t *restrict cond,const pthread_condattr_t *restrict attr);
    初始化
int pthread_cond_destroy(pthread_cond_t *cond);
    释放
int pthread_cond_wait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex);
    等待唤醒
int pthread_cond_timedwait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex,const struct timespec *restrict abstime);
    等待，依据传入的时间 设置阻塞时间
int pathread_cond_signal(pthread_cond_t *cond);
    唤醒一个或者多个等待线程
int pthread_cond_broadcast(pthead_cond_t *cond);
    唤醒全部线程
*/

/*
生产者消费者模型
*/

#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

struct Node
{
    int num;
    struct Node * next;
};
//头结点
struct Node * head = NULL;
//定义互斥锁
pthread_mutex_t mutex;
//创建条件变量
pthread_cond_t cond;

void * producer(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        struct Node * newNode = (struct Node *)malloc(sizeof(struct Node));
        newNode->next = head;
        head = newNode;
        newNode->num = rand()%1000;
        printf("add node,num : %d,tid : %ld \n",newNode->num,pthread_self());
        //只要生产了一个就通知消费者
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
        usleep(100);
    }
    return NULL;
}

void * customer(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        struct Node * temp = head;
        if(head!=NULL){
            head = head->next;
            printf("del node num : %d, tid : %ld\n",temp->num,pthread_self());
            temp->next = NULL;
            free(temp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        }else{
            //释放
            pthread_mutex_unlock(&mutex);
            //没有数据等待
            //当这个函数调用阻塞的时候，会释放锁，生产者开始生成
            //但不阻塞的时候，继续向下执行，不阻塞
           pthread_cond_wait(&cond,&mutex);
        }
    }
    return NULL;
}


int main(){
    //创建互斥锁
    pthread_mutex_init(&mutex,NULL);
    //初始化cond
    pthreaD_cond_init(&cond,NULL);
    //创建5个生产，5个消费
    pthread_t ptids[5];
    pthread_t ctids[5];
    for(int i=0;i<5;i++){
        pthread_create(&ptids[i],NULL,producer,NULL);
    }

    for(int i=0;i<5;i++){
        pthread_create(&ctids[i],NULL,customer,NULL);
    }
    //线程连接
    for(int i=0;i<5;i++){
        pthread_join(ptids[i],NULL);
        pthread_join(ctids[i],NULL);
    }
    // while(1){
    //     sleep(10);
    // }
    //
    //消除互斥锁
    pthread_mutex_destroy(&mutex);
    //消除条件变量
    pthread_cond_destroy(&cond);
    pthread_exit(NULL);

    return 0;
}
```

---

##### 信号量
不可保证线程安全。

```c++
//信号量的类型 sem_t
int sem_init(sem_t *sem,int pshared,unsigned int value);
int sem_destroy(sem_t * sem);
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem)
int sem_timedwait(sem_t *sem,const struct timespec *bas_timeout);
int sem_post(sem_t *sem);
int sem_getvalue(sem_t *sem,int *sval);
```
