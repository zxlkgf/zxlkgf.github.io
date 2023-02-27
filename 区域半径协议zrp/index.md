# 区域半径协议ZRP

# ZRP(区域半径协议)详解

---

## 1.默认配置参数Constants.h

```cpp
#ifndef _constants_h_
#define _constants_h_

// General OR Common Constants
//DEBUG 参数
#define DEBUG                0    // 1 = Enabled; 0 = Disabled
//Router_PORT
#define ROUTER_PORT          0xff
#define TRUE                 1
#define FALSE                 0
#define LINKUP                 1
#define LINKDOWN             0
#define DEFAULT_STARTUP_JITTER         1    // 1sec
#define DEFAULT_EXPIRATION_CHECK_PERIOD 1   // 1sec... 
#define MAX_SEQUENCE_ID         1000000 // For Packet Utils
#define MIN_PACKET_DROP_TIME        3    // 3 sec For 1-hop communication    [Not Used]
#define MAX_PACKET_DROP_TIME        10    // 10 sec For multi-hop communication    [Not Used]
#define IERP_TTL            50    // Maximum-length for IERP route

// NDP related Constants
#define DEFAULT_BEACON_PERIOD         3     // 3 sec
#define DEFAULT_BEACON_PERIOD_JITTER    1     // 1 sec
#define DEFAULT_NEIGHBOR_ACK_TIMEOUT      2     // 2 sec  [Within this much time ACK should come to me]
#define DEFAULT_MAX_ACK_TIMEOUT        2    // How many ACK Timeout is needed to declare a Neighbor DOWN

// IARP related Constants
#define DEFAULT_MIN_IARP_UPDATE_PERIOD     3     // 3 sec  [T_lsu]
#define DEFAULT_MAX_IARP_UPDATE_PERIOD     10     // 10 sec [T_lsu]
#define DEFAULT_LINK_LIFETIME         10    // 10sec    
#define DEFAULT_UPDATE_LIFETIME     30    // 30sec  [For Control Flooding]

// IERP related Constants
#define DEFAULT_BRP_XMIT_POLICY        0    // 1: BRP_MULTICAST, 0: BRP_UNICAST
#define IERP_REPLY_SNOOP        1    // 1: Enabled, 0: Disabled
#define IERP_ERROR_SNOOP        1    // 1: Enabled, 0: Disabled
#define IERP_XMIT_JITTER        1    // 1sec [Uniformly distributed]
#define DEFAULT_QUERY_LIFETIME         30     // 30sec  [For Control Flooding]
#define DEFAULT_QUERY_RETRY_TIME    5    // 5sec
#define DEFAULT_ROUTE_LIFETIME         120    // 120sec  [IERP Route Reliability]
#define DEFAULT_MAX_IERP_REPLY        3    // Destination can send maximum 5 replies...

// ZRP related Constants
#define ZRP_DEFAULT_HDR_LEN        10    // 10 bytes [refer to hdr_zrp struct]
#define DEFAULT_ZONE_RADIUS         2
#define DEFAULT_SEND_BUFFER_SIZE    100    // 100 Packets
#define DEFAULT_PACKET_LIFETIME        20    // 10 sec
#define    DEFAULT_INTERPKT_JITTER        1    // 1sec [Uniformly distributed]

// Type-Defintions & Enumeration...
typedef double Time;   
typedef int32_t Query_ID;

// Types of ZRP packets...
enum ZRPTYPE {
      NDP_BEACON, NDP_BEACON_ACK, IARP_UPDATE, IARP_DATA, IERP_REPLY, IERP_REQUEST, IERP_ROUTE_ERROR, IERP_DATA
};

// BRP Xmit Policy...
enum BRP_XMIT_POLICY {
    BRP_UNICAST, BRP_MULTICAST
};

#endif
```

---

## 2.NDP(邻居发现协议)

---

### 2.1 NDPAgent的类

```cpp
/* [SUB-SECTION-2.4]------------------------------------------------------------
 * NDP AGENT:- NEIGHBOR DISCOVERY AGENT
 * NDP AGENT:- 邻居发现代理类
 * -----------------------------------------------------------------------------
 */
/*!    \class NDPAgent
    \brief This class implements the Neighbor Discovery Protocol (NDP).
 */
class NDPAgent
{
  public:
    // Data...
    ZRPAgent *agent_;        // 指向ZRPAgent的指针
    NeighborList neighborLst_;    // 存储邻居信息的邻居链表
    NDPBeaconTransmitTimer BeaconTransmitTimer_;    // 发送邻居探索包的类
    NDPAckTimer AckTimer_;                // 发送包等待邻居接受并返回ACK的类，如果邻居超过时间没有返回ACK则处理邻居
    int startup_jitter_;        // 启动的时间

    // 构造方法...
    /*!    \fn NDPAgent(ZRPAgent *)
        This constructor is called by the constructor of ZRP. It initializes 
        different compenents in NDP.
        这个构造方法在ZRP的构造方法中被调用。
     */
    NDPAgent(ZRPAgent *agent) : agent_(agent), BeaconTransmitTimer_(agent),
        AckTimer_(agent), startup_jitter_(DEFAULT_STARTUP_JITTER) {}

    // Methods...
    /*!    \fn void startUp()
        This is called by ZRPAgent::startUp() method. It does following tasks: \n
        - Starts the Beacon-Transmit-Timer. \n
        - Clears the Neighbor-List.
        这个方法在ZRPAgent类的StartUp方法中被调用
        并执行开启Beacon-Transmit-Timer
        以及清空邻居节点链路表
    */
    void startUp();

    /*!    \fn void recv_NDP_BEACON(Packet* p)
        This function is called whenever ZRP receives an NDP_BEACON packet. On receving 
        'NDP_BEACON', node sends 'NDP_BEACON_ACK' to the sender. But it does not add this 
        neighbor, because node stores only symmetric links.
        每当当前节点的ZRP代理人接收到NDP_BEACON packet，就会想发送这个NDP_BEACON packet的节点
        发送NDP_ACK packet
    */
    void recv_NDP_BEACON(Packet* p);

    /*!    \fn void recv_NDP_BEACON_ACK(Packet* p)
        This function is called whenever ZRP receives an NDP_BEACON_ACK packet. \n
        - If it's a new neighbor, node adds the neighbor to its 'NEIGHBOR_TABLE' and 
        notifies IARP to rebuild the 'INNER_ROUTING_TABLE' and 'PERIPHERAL_NODE_TABLE'. \n
        - If it's an existing neighbor, it updates 'LAST_ACK_TIME' for that neighbor.
        每当当前节点的ZRP代理类接收到NDP_BEACON_ACK packet的时候，就会判断当前发送这个包的节点
        是不是新邻居节点，如果是新邻居节点，就将它加入到NEIGHBOR_TABLE邻居表中，并通知IARPAgent
        代理人去重构它的INNER_ROUTING_TABLE和PERIPHERAL_NODE_TABLE。
        如果是已经存在的邻居节点，则更新他的LAST_ACK_TIME
    */
    void recv_NDP_BEACON_ACK(Packet* p);
    /*!    \fn void print_tables()
        This function prints the Neighbor-List. It also prints which Neighbors are UP 
        and which are DOWN.
        这个方法打印邻居节点的链表，并且哪个邻居节点处于linked-up，哪个节点处于linked-down
    */
    void print_tables();
};
```

---

### 2.1.1 NDPAgent类内成员NeighborList类(邻居节点表类)

该表维护一个NeighborList(邻居节点表)

```cpp
/*!    \class NeighborList
    \brief This class is the linked-list of Neighbor class.
 */
class NeighborList
{
  public:
    Neighbor *head_;    // 指向链表头部的指针
    int numNeighbors_;    // 链表内成员数量

    // 构造方法...
    NeighborList() : head_(NULL), numNeighbors_(0) {}

    // 方法...
    // 将邻居节点加入
    void addNeighbor(nsaddr_t addr, Time lastack, int linkStatus);
    // 查找
    int findNeighbor(nsaddr_t addr, Neighbor **handle);
    // 判空
    int isEmpty();
    // 移除
    void removeNeighbor(Neighbor *prev, Neighbor *toBeDeleted);
    // 移除所有链路状况为down的邻居节点
    void purgeDownNeighbors();
    // 清空链表
    void freeList();
    // 打印链表
    void printNeighbors();    
};
```

---

#### 2.1.1.1 NeighborList类内的成员类Neighbor类

```cpp
/*
* 该类存储邻居节点的详细信息
*/
class Neighbor
{
  public:        
    nsaddr_t addr_;        // 32 位的地址
      Time lastack_;        // 上一次接收到ACK的时间
    int linkStatus_;    // 链路状况
    int AckTOCount_;    // 超时的次数
    Neighbor *next_;    // next指针

    // 构造方法...
    Neighbor() : addr_(-1), lastack_(-1), linkStatus_(LINKDOWN),
        next_(NULL) {}    // Initialize with Invalid Entries
    Neighbor(nsaddr_t addr, Time lastack, int linkStatus) : addr_(addr),
        lastack_(lastack), linkStatus_(linkStatus), AckTOCount_(0),
        next_(NULL) {} 
};
```

---

#### 2.1.1.2 NeighborList类的addNeighbor()的实现

```cpp
/* 
* 将单个邻居节点加入邻居链表中
* nsaddr_t addr:  32 位地址
* Time lastack:   接收ACK的时间
* int linkStatus: 链路状态
*/
void
NeighborList::addNeighbor(nsaddr_t addr, Time lastack, int linkStatus) {
    //创建新的邻居节点
    Neighbor *newNb = new Neighbor(addr, lastack, linkStatus);
    if(newNb == NULL) {    // 检查创建情况
        printf("### Memory Allocation Error in [NeighborList::addNeighbor] ###");
        exit(0);
    }
    newNb->next_ = head_;    // 头插法插入    
    head_ = newNb;
    numNeighbors_++;    // ++链表内节点数量
}
```

---

#### 2.1.1.3 NeighborList类的findNeighbor()的实现

```cpp
/* 
* 查找邻居链表内的是否存在当前邻居节点
* 如果有 返回true 并且将其赋值给handle    
* 如果没有 返回false
* nsaddr_t addr : 32位地址
* Neighbor **handle : handle指针
 */
int
NeighborList::findNeighbor(nsaddr_t addr, Neighbor **handle) {
    // 定义指向NeighborList的head的临时指针
    Neighbor *cur = NULL;
    // 查找的返回值
    int foundFlag; 
    // 小心空链表
    // [i<numNeighbors_] condition takes care for EMPTY List case...
    cur=head_;//头节点赋值给临时变量
    foundFlag=FALSE;//先赋值为false
    //遍历循环查找节点
    for(int i=0; i<numNeighbors_; i++) {
        if(addr == cur->addr_) {
            foundFlag = TRUE;
            break;
        }
        cur=cur->next_;
    }
    if(foundFlag) {        // 如果邻居节点存在
        *handle = cur;    // 将该邻居节点的地址赋值给handle
        return TRUE;    // 返回true        
    }

    return FALSE;        // 邻居未找到 返回false
}
```

---

#### 2.1.1.4 NeighborList类的isEmpty()实现

```cpp
/* 
* 如果为空返回true
* 如果为空返回false
*/
int
NeighborList::isEmpty() {
    if(numNeighbors_ == 0) {
        return TRUE;
    } else {
        return FALSE;
    }
}
```

---

#### 2.1.1.5 NeighborList类的removeNeighbor()的实现

```cpp
/* Remove a Neighbor specified by the pointer in argument. 
* 移除参数中指定的邻居节点
* Neighbor *prev: 被删除节点的前置节点
* Neighbor *toBeDeleted: 被删除节点
*/
void 
NeighborList::removeNeighbor(Neighbor *prev, Neighbor *toBeDeleted) {
    // [Case-1]: This is a head that needs to be deleted
    // 如果前置节点为空，并且被删除的节点不是头节点，则报错退出
    if(prev == NULL ) {    
        if(toBeDeleted != head_) {
            printf("### Logical Error in [NeighborList::removeNeighbor] ###");
            exit(0);
        }
        //否则，删除节点就是头节点
        head_ = head_->next_; // 让head指针 指向head的next指针
        delete toBeDeleted;  // 删除节点空间
        numNeighbors_--;    //邻居表内成员--
        return;
    }

    // [Case-2]: 通常的case
    prev->next_ = toBeDeleted->next_; // 前节点的next指向删除节点的next
    delete toBeDeleted;    //删除想要删除的节点
    numNeighbors_--;    //邻居表内成员--
}
```

---

#### 2.1.1.6 NeighborList类的purgeDownNeighbors()的实现

```cpp
/* Remove all Neighbors for which the linkStatus_ is DOWN. 
* 删除所有链路状态位DOWN的节点
*/
void 
NeighborList::purgeDownNeighbors() {
    //如果邻居表为空
    if(numNeighbors_ == 0) {
        return;        // Nothing to do
    }
    // [case-1]: Leaving the 1st case(head_ case)
    // 
    Neighbor *prev, *cur, *toBeDeleted;
    prev = head_;
    cur = head_->next_;
    //循环遍历，只要下一个节点不为空
    for(;cur!=NULL;) {
        // 如果下一个节点的链路状态位LINKDOWN
        if(cur->linkStatus_ == LINKDOWN) {    // Delete the Neighbor
            prev->next_ = cur->next_;//前置节点的next指向被删除节点的next
            toBeDeleted = cur;//cur赋值给需要删除的节点的指针
            cur = cur->next_;//cur指向下一个节点
            delete toBeDeleted;//删除被删除节点的空间
            numNeighbors_--;//邻居表内成员--
        } else {
            prev = cur;        // 前节点等于下一个节点
            cur = cur->next_;//下一个节点指向下一个节点的next                
        }
    }
    // [case-2]: head_ case...
    // 头节点特判
    if(head_!=NULL) {
        //如果头节点为LINKDOWN
        if(head_->linkStatus_ == LINKDOWN) {
            toBeDeleted = head_; //赋值给需要删除的节点的指针
            head_ = head_->next_;//头节点指向头节点的next
            delete toBeDeleted;//删除
            numNeighbors_--;//邻居表内成员--
        }
    }
}
```

---

#### 2.1.1.7 NeighborList类的void freeList()的实现

```cpp
/*  
* 删除整个表
*/
void 
NeighborList::freeList() {
    //表内元素为空
    if(numNeighbors_ == 0) {
        return;
    }
    //指定临时变量
    Neighbor *toBeDeleted, *cur = NULL;    
    // [i<numNeighbors_] condition takes care for EMPTY List case...
    cur = head_;
    for(int i=0; i<numNeighbors_; i++) {
        toBeDeleted = cur;
        cur = cur->next_;
        delete toBeDeleted;
    }
    head_ = NULL;        // Reset the attributes...
    numNeighbors_ = 0;    // Reset the attributes...
}
```

---

#### 2.1.1.8  NeighborList类的printNeighbors()的实现

```cpp
/* Print all Neighbors to the console. 
* 控制台打印邻居链表内所有邻居信息
*/
void
NeighborList::printNeighbors() {
    // Print current Neighbor-Table
    printf("Neighbors:[");
    if(numNeighbors_ == 0) {
        printf("EMPTY]");
        return;    
    }
    Neighbor *curNb = NULL;
    curNb = head_;
    // 打印链路状态为up的邻居节点
    printf("(Up: ");
    for(int i=0; i<numNeighbors_; i++) {
        if(curNb->linkStatus_ == LINKUP) {
            printf("%d ",curNb->addr_);
        }
        curNb = curNb->next_;
    }
    //打印链路状态为DOWN的邻居节点
    printf("),(Down: ");
    curNb = head_;
    for(int i=0; i<numNeighbors_; i++) {
        if(curNb->linkStatus_ == LINKDOWN) {
            printf("%d ",curNb->addr_);
        }
        curNb = curNb->next_;
    }
    printf(")]");
}
```

---

### 2.1.2 NDPAgent类内成员NDPBeaconTransmitTimer类

该类定时发送邻居节点的探测包

```cpp
/* [SUB-SECTION-2.2]------------------------------------------------------------
 * BEACON-TRANSMIT TIMER:- PERIODIC BEACON XMISSION
 * -----------------------------------------------------------------------------
 */
/*!    \class NDPBeaconTransmitTimer
    \brief This class implements a BEACON Transmit Timer, 
    \who periodically sends beacons to NEIGHBOR nodes.
 */
class NDPBeaconTransmitTimer : public Handler 
{
  public:
    // Data...
    ZRPAgent *agent_;        // 指向ZRPAgent的指针
    Event intr_;        
    int beacon_period_;         // 发包时间间隔
    int beacon_period_jitter_;    // Jitter added to Inter-Beacon-Time
    int neighbor_ack_timeout_;     // Only for Logging为了日志打印

    // 构造器...
    NDPBeaconTransmitTimer(ZRPAgent* agent) : agent_(agent),
        beacon_period_(DEFAULT_BEACON_PERIOD), 
        beacon_period_jitter_(DEFAULT_BEACON_PERIOD_JITTER),
        neighbor_ack_timeout_(DEFAULT_NEIGHBOR_ACK_TIMEOUT) {}

    void handle(Event*);          // function handling the event
    void start(double thistime);
};
```

---

#### 2.1.2.1 NDPBeaconTransmitTimer类的start()的实现

```cpp
/* Starts BeaconTransmitTimer, called by startUp(), delayed by 'thistime'. 
* 被NDPAgent的startUp方法调用
* thistime:为延迟时间
*/
void 
NDPBeaconTransmitTimer::start(double thistime) {
    //将其提交给ns2的Scheduler去计划启动
    Scheduler::instance().schedule(this, &intr_, thistime );
}
```

---

#### 2.1.2.1 NDPBeaconTransmitTimer类的handle()的实现

```cpp
/* Broadcasts a Beacon. 
* 广播邻居查找
*/
void 
NDPBeaconTransmitTimer::handle(Event* e) {
    // [Task-1]: 创建一个广播信标
    // 包的指针
    Packet* p = NULL;
    //调用方法创建一个NDP_BEACON,IP为IP_BROADCAST,TIME TO LIVE = 1
    p = (agent_->pktUtil_).pkt_create(NDP_BEACON, IP_BROADCAST, 1); // last arg is TTL=1
    //调用pktUtil_的广播方法
    (agent_->pktUtil_).pkt_broadcast(p, 0.00);             // broadcast pkt
                    if(DEBUG) { // [Log: XMIT_NDP_BEACON]
                    Time now = Scheduler::instance().clock(); // get the time
                    hdr_zrp *hdrz = HDR_ZRP(p);           // access ZRP part of pkt header
                    printf("\n_%d_ [%6.6f] | XMIT_NDP_BEACON | -S %d -Nb BROADCAST | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->seq_);
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
    // [Task-2]: 开启ACKTimer
    (agent_->ndpAgt_).AckTimer_.start();
    // [Task-3]: Schedule next scan in BEACON_PERIOD + Jitter sec
    // 将其交给Schedule，延迟时间BEACON_PERIOD + Jitter sec之后再次调用
      Scheduler::instance().schedule(this, &intr_, beacon_period_ + Random::uniform(beacon_period_jitter_));
}
```

---

### 2.1.3 NDPAgent类内成员NDPAckTimer类

```cpp
/* [SUB-SECTION-2.3]------------------------------------------------------------
 * ACK-TIMEOUT TIMER:- ACK-TIMEOUT CHECKING
 * -----------------------------------------------------------------------------
 */
/*!    \class NDPAckTimer
    \brief This class implements a Acknoledgement TimeOut Timer, who checks
           that ACK is received or not for sent beacon.
    \该类为一个超时类，检查有没有接收到ACK
 */
class NDPAckTimer : public Handler 
{
  public:
    ZRPAgent *agent_;        // 指向ZRP-Agent的指针
    Event intr_;
    int neighbor_ack_timeout_;    // ACK送信超时时间    

    // Constructor...
    NDPAckTimer(ZRPAgent* agent) : agent_(agent),
        neighbor_ack_timeout_(DEFAULT_NEIGHBOR_ACK_TIMEOUT) {}

      void handle(Event*);          // function handling the event
    void start();
};
```

---

#### 2.1.3.1 NDPAckTimer类方法start()的实现

```cpp
/* [SUB-SECTION-1.3]--------------------------------------------------------------------------------------------------------
 * ACK-TIMEOUT TIMER:- ACK-TIMEOUT CHECKING
 * -------------------------------------------------------------------------------------------------------------------------
 */
/* Schedule timeout at neighbor_ack_timeout_, 
* just after sending a Beacon. 
* 在发送NDP Beacon之后启动
*/
void 
NDPAckTimer::start() {
    Scheduler::instance().schedule(this, &intr_, neighbor_ack_timeout_);
}
```

---

#### 2.1.3.2 NDPAckTimer类方法handle()的实现

```cpp
/* 
* Mark LINKDOWN for un-ACKed neighbors. 
* If any of them found DOWN then notify IARP to rebuild routing table. 
* 为未确认ACK的邻居设置LinkStatus为LinkDOWN
* 一旦发现LinkDown的邻居节点，就通知IARP重新创建路径表
*/
void 
NDPAckTimer::handle(Event* e) {
    // [Task-1]: Check if Neighbor Table is Empty...
    //检查邻居链表是否为空
    if((agent_->ndpAgt_).neighborLst_.isEmpty()) {
                    if(DEBUG) { // [Log: NDP_ISOLATED_NODE ]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | NDP_ISOLATED_NODE | No NDP_BEACON_ACK Detected | ",
                    agent_->myaddr_, now); agent_->print_tables(); printf("\n");
                    } // [Log: End]
        return;//什么都不做
    }
    // [Task-2]: Drop timed-out neighbors & inform IARP about it...
    // 删除超时的邻居节点，并且通知IARP
    Time now = Scheduler::instance().clock(); // 获取时间
    Neighbor *prev, *curNb; //定义存储邻居节点的前置指针和临时指针
    prev = NULL;    // 必须置为 NULL
    //指向当前ZRPAgent的NDPAgent下的邻居表的头节点
    curNb = (agent_->ndpAgt_).neighborLst_.head_;
    //循环检查链表内邻居节点的ACK时间
    for(; curNb!=NULL; ) {
        // [SubTask-2.1]: Check for Ack Timeout of Neighbor
        // 如果当前时间-邻居节点的最近一次时间超过Ack超时时间
        if((now - curNb->lastack_) > neighbor_ack_timeout_) {
        // 检查当前节点的超时次数
        // [SubTask-2.2]: Check for How many TimeOuts have occured with this Neighbor
            //超时次数先++
            curNb->AckTOCount_++;
            //检查是否超多最大超时次数
            if(curNb->AckTOCount_ >= DEFAULT_MAX_ACK_TIMEOUT) {
        // 将该节点的LinKStates设置为DOWN，并且通知IARP
        // [SubTask-2.3]: Detected a DOWN Neighbor - Notify IARP & Take appropriate actions     
                //将IARP内部的updateSendFlag_设置为true,为下次更新做准备
                (agent_->iarpAgt_).updateSendFlag_ = TRUE; // Set the Update at Next Timer-Event 
                //定义LinkState的临时handleToFoundLS
                LinkState *handleToFoundLS = NULL;
                //定义查找flag
                int foundFlag;
                //从当前ZRPAgent的IARPAgent的LinKState链表内查询
                foundFlag = (agent_->iarpAgt_).lsLst_.findLink(curNb->addr_, agent_->myaddr_, &handleToFoundLS);
                //如果查到了
                if( foundFlag == TRUE ) { 
                    handleToFoundLS->isup_ = LINKDOWN;      // 将link-stauts设置为DOWN 
                    (agent_->iarpAgt_).buildRoutingTable();      // 重构 Routing Table
                }
        // [SubTask-2.4]: Set the Link-Status of Neighbor to LINKDOWN...
        //          [DOWN Neighbors are deleted after sending IARP_UPDATE - I didn't forget that :) ]
                //将该当邻居节点的linkStatus设置为DOWN
                curNb->linkStatus_ = LINKDOWN;
            }
                    if(DEBUG) { // [Log: NDP_BEACON_ACK_TIMEOUT]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | NDP_BEACON_ACK_TIMEOUT | -S %d -Nb %d | -TimeOut %d | ",
                    agent_->myaddr_, now, agent_->myaddr_, curNb->addr_, neighbor_ack_timeout_);    
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]        
        }
        //继续遍历
        prev = curNb;        // Advance the Pointers    
        curNb=curNb->next_;
    }
}
```

---

### 2.1.4 NDPAgnet类内方法startUp()

```cpp
/* [SUB-SECTION-1.4]--------------------------------------------------------------------------------------------------------
 * NDP AGENT:- NEIGHBOR DISCOVERY AGENT
 * -------------------------------------------------------------------------------------------------------------------------
 */
/*开启邻居节点信标探测，清空链表 */
/*!    \fn void startUp()
        This is called by ZRPAgent::startUp() method. It does following tasks: \n
        - Starts the Beacon-Transmit-Timer. \n
        - Clears the Neighbor-List.
        这个方法在ZRPAgent类的StartUp方法中被调用
        并执行开启Beacon-Transmit-Timer
        以及清空邻居节点链路表
*/
void
NDPAgent::startUp() {
    // [Task-1]: Start the Timers... [We donot need to start ACK timer- It is started by BTTimer]
    // 开启BeaconTransitTimer 我们不需要开启ACKtimer 他在BeaconTransmitTimer_之后被启动
    double startUpJitter;
      startUpJitter =  Random::uniform(startup_jitter_);
    BeaconTransmitTimer_.start(startUpJitter);    

    // [Task-2]: Clear the NeighborList...
    // 清空邻居链表
    if(!neighborLst_.isEmpty()) {
        neighborLst_.freeList();
    }
}
```

---

### 2.1.5 NDPAgnet类内方法recv_NDP_BEACON(Packet* p)

在ZRPAgent::recv()方法中被使用

```cpp
/* 接收到NDP_BEACON的时候，返回NDP_BEACON_ACK*/
void
NDPAgent::recv_NDP_BEACON(Packet* p) {
    // [Assumption]:We are looking only for symmetric links. 
    // [Assumption]:我们只寻找对称链接 
    // On receving NDP_BEACON, I donot consider the sender as a Neighbor.
    // 当接收到NDP_BEACON的时候，我们不讨论他是否是邻居
                    if(DEBUG) { // [Log: RECV_NDP_BEACON]
                    Time now = Scheduler::instance().clock(); // get the time
                    hdr_zrp *hdrz = HDR_ZRP(p);         // access ZRP part of pkt header
                    printf("\n_%d_ [%6.6f] | RECV_NDP_BEACON | -S %d -Nb %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, agent_->myaddr_, hdrz->seq_);    
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
    // [Task-1]: Create an ACK packet and Send it...
    //  创建ACK包并且发送
    Packet *pnew = NULL;//创建新的包的临时变量
      hdr_zrp *hdrz = HDR_ZRP(p);//从接收到的包里分离出zrp的包头
    //创建包，类型NDP_BEACON_ACK，目标为发送节点，TIME TO LIVE = 1
    pnew = (agent_->pktUtil_).pkt_create(NDP_BEACON_ACK, hdrz->src_, 1);
    // Unicast the packet (单播包)
    (agent_->pktUtil_).pkt_send(pnew, hdrz->src_, 0.00);
                    if(DEBUG) { // [Log: XMIT_NDP_BEACON_ACK]
                    Time now = Scheduler::instance().clock(); // get the time
                    hdr_zrp *hdrznew = HDR_ZRP(pnew);     // access ZRP part of pkt header
                    printf("\n_%d_ [%6.6f] | XMIT_NDP_BEACON_ACK | -S %d -Nb %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrznew->dest_, hdrznew->src_, hdrznew->seq_);    
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
    // [Task-2]: Drop the NDP_BEACON just Received...
    //丢弃掉已经接收到的NDP_BEACON
    (agent_->pktUtil_).pkt_drop(p);
}
```

---

### 2.1.6 NDPAgnet类内方法recv_NDP_BEACON_ACK(Packet* p)

```cpp
/* On receiving ACK, Update the Neighbor-table and IARP-(topology & routing)-table if required. */
//收到 ACK 后，如果需要，更新邻居表和 IARP（拓扑和路由）表
void 
NDPAgent::recv_NDP_BEACON_ACK(Packet* p) {
    // [Task-1]: Check if Neighbor exists - If not, add it & Notify IARP...
    Time now = Scheduler::instance().clock();     // 获取当前时间
    hdr_zrp *hdrz = HDR_ZRP(p);// 获取ZRP的包头
    Neighbor *handleToFoundNb = NULL;//定义查找邻居节点的指针
    //在邻居节点内部查找该邻居是否存在
    int foundNeighbor = neighborLst_.findNeighbor(hdrz->src_, &handleToFoundNb);
    //如果该邻居存在
    if(foundNeighbor == TRUE) {
                    if(DEBUG) { // [Log: RECV_NDP_BEACON_ACK]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_NDP_BEACON_ACK | -S %d -Nb %d | -SEQ %d | "
                    "Existing Neighbor Detected | ",
                    agent_->myaddr_, now, hdrz->dest_, hdrz->src_, hdrz->seq_);    
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
        //将其最后一次ack设置为当前时间
        handleToFoundNb->lastack_ = now;
        //将其linkStatus_状态设置为UP    
        handleToFoundNb->linkStatus_ = LINKUP;
        //将其ACK未确认次数归零
        handleToFoundNb->AckTOCount_ = 0;
    } else { // 如果为查找到，意味着发现新邻居    
                    if(DEBUG) { // [Log: RECV_NDP_BEACON_ACK]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_NDP_BEACON_ACK | -S %d -Nb %d | -SEQ %d | "
                    "New Neighbor Detected | ",
                    agent_->myaddr_, now, hdrz->dest_, hdrz->src_, hdrz->seq_);    
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
        //在邻居表中添加新邻居信息
        neighborLst_.addNeighbor(hdrz->src_, now, LINKUP);
        // Notify IARP to send the next Update [Change of Neighbor-Table Detected]
        //通知IARP 在下次更新的时候更新路径表
         (agent_->iarpAgt_).updateSendFlag_ = TRUE; // Set the Update at Next Timer-Event    
    }

    // [Task-2]: Update IARP...    
    // 更新IARP
    // 定义LinkState的临时存储指针
    LinkState *handleToFoundLS = NULL;
    //查询Link状态
    int LSFoundFlag = (agent_->iarpAgt_).lsLst_.findLink(hdrz->src_, agent_->myaddr_, &handleToFoundLS);
    if( LSFoundFlag == FALSE ) { 
        //如没查询到，则加入该链路信息
        (agent_->iarpAgt_).lsLst_.addLink(hdrz->src_, agent_->myaddr_, hdrz->seq_, 
                            LINKUP, now+(agent_->iarpAgt_).linkLifeTime_);
        (agent_->iarpAgt_).buildRoutingTable();      // 重构 Routing Table
        //从 Send_Buffer 中查找有无需要发送的数据包
        agent_->route_SendBuffer_pkt();    
    } else {    // 如果链路查找到了，更新其内属性
        handleToFoundLS->isup_ = LINKUP;
        handleToFoundLS->expiry_ = now+(agent_->iarpAgt_).linkLifeTime_;
    }

    // [Task-3]: 丢弃该包
    (agent_->pktUtil_).pkt_drop(p);
}
```

---

## 3. IARP(主动协议)

---

### 3.1 IARPAgent

ZRP协议内部维护区域半径的主动协议

```cpp
/* [SUB-SECTION-3.7]------------------------------------------------------------
 * IARP AGENT:- INTRA-ZONE ROUTING AGENT
 * -----------------------------------------------------------------------------
 */
/*!    \class IARPAgent
    \brief This class implements the IntrAzone Routing Protocol (IARP).
    \该类包含了IARP协议
 */
class IARPAgent
{
  public:
    // Data...
    ZRPAgent *agent_;    // 指向ZRPAgent的指针
    int updateSendFlag_;// 初期化为TRUE [定期更新被发送
                // 如果它为True,每次更新都被发送
                // 如果它为False
    // 表...
    LinkStateList lsLst_;    // 拓扑表
    PeripheralNodeList pnLst_;    //外围节点表
    InnerRouteList irLst_;    //主动路径表
    IARPUpdateDetectedList upLst_;    // 检测到更新缓存
    int linkLifeTime_;    // DEFAULT_LINK_LIFETIME 
    int updateLifeTime_;// DEFAULT_UPDATE_LIFETIME

    // Timers...
      IARPPeriodicUpdateTimer PeriodicUpdateTimer_;//定期更新计时器
    IARPExpirationTimer ExpirationTimer_; //定期判断超时器
    int startup_jitter_;    // For starting the Timers

    // 构造器...
    /*!    \fn IARPAgent(ZRPAgent *)
        该构造函数由ZRP的构造函数调用。它初始化IARP 中的不同组件。
     */
    IARPAgent(ZRPAgent *agent) : agent_(agent), updateSendFlag_(TRUE),
        linkLifeTime_(DEFAULT_LINK_LIFETIME),
        updateLifeTime_(DEFAULT_UPDATE_LIFETIME), 
        PeriodicUpdateTimer_(agent), ExpirationTimer_(agent),
        startup_jitter_(DEFAULT_STARTUP_JITTER) {}

    // 方法...
    /*!    \fn void startUp()    
        这是由 ZRPAgent::startUp() 方法调用的。它执行以下任务: \n
        - 启动 Periodic-Update-Timer 和 Expiration-Timer。
        - 清除 LinkState-List、PeripheralNode-List、Inner-Route-List &
          Detected-Update-List.
    */
    void startUp(); // 启动Timers &清空Lists(Tables)
    /*!    \fn void buildRoutingTable()
        该函数基于创建Link-State-List(链路状况表)来创建
        Inner-Route-List(内部路由路径表) & Peripheral-Node-List(外围节点表)
        执行以下步骤。
        -# 清空已存在的 Inner-Route-List & Peirpheral-Node-List.
        -# 清除 expired(过期) links & DOWN links.
        -# 从现有链路状态列表生成 BFS 树.
        -# 基于该 BFS 树生成内部路由列表。插入所有叶子外围节点列表中的节点。
    */
    void buildRoutingTable();// 创建基于Topology-Table 的 Routing-Table
    /*!    \fn void addRouteInPacket(nsaddr_t dest, Packet *p)
        This function adds the Inner-route for node 'dest' in packet 'p' if available.
        该方法判断数据包P是否可用，如果可用，则将其添加到inner-route
    */
    void addRouteInPacket(nsaddr_t dest, Packet *p);
    /*!    \fn void recv_IARP_UPDATE(Packet* p)
        只要 ZRP 收到 IARP_UPDATE 数据包，就会调用此函数。
        执行以下任务-
        - 如果是一个新的iarp_update那么,
            - 节点更新它自己的 'LINK_STATE_TABLE',
            - 如果 'LINK_STATE_TABLE' 被改变，那么, 就重构以下两个 
              'INNER_ROUTING_TABLE' and 'PERIPHERAL_NODE_TABLE',
            - 将此更新缓存在“UPDATE_DETECT_TABLE”中以控制泛洪，
            - 如果 TTL 不等于 0，则重新广播它.
        - 如果它是一个已经收到的 iarp_update 那么，
        - 节点丢弃此更新
    */
    void recv_IARP_UPDATE(Packet* p); // 接收IARP更新信息
    /*!    \fn void recv_IARP_DATA(Packet* p)
        只要 ZRP 收到 IARP_DATA 数据包，就会调用此函数。
        执行以下任务 -    
        - 如果该包的目标节点与当前节点相匹配，想消息发送到上一层
        - 否则
        - 转发该包
    */
    void recv_IARP_DATA(Packet* p);    // Receiving IARP DATA(Local Traffic)
    /*!    \fn void print_tables()
        该方法打印Peripheral-Node-List & Inner-Route-List.
    */
    void print_tables();        // Print all Tables
};
```

---

#### 3.1.1 IARPAgent类内成员LinkStateList类

```cpp
//该类维护一个拓扑链表
class LinkStateList
{
  public:
    // Data...
    LinkState *head_; //表头
    int numLinks_;     //内部成员数量

    // 构造方法...
    LinkStateList() : head_(NULL), numLinks_(0) {}

    // 方法...
    void addLink(nsaddr_t src, nsaddr_t dest, int seq, int isup, Time expiry);
    int findLink(nsaddr_t src, nsaddr_t dest, LinkState **handle);
    int isEmpty();
    void removeLink(LinkState *prev, LinkState *toBeDeleted);
    int purgeLinks();
    void printLinks();
    void freeList();
};
//该类维护单个拓扑信息
class LinkState
{
  public:
    // Data...
      nsaddr_t src_;      // 32 bits的源节点
      nsaddr_t dest_;      // 32 bits的目标节点
      int seq_;          // sequence number
      int isup_;        // Link State LINKUP[1]/LINKDOWN[0]（状态）
      Time expiry_;        // Link Expiry Time    (更新时间)
    LinkState *next_;    // Pointer to Next Element （下一个指针）

    // 构造方法...
    LinkState() : src_(-1), dest_(-1), seq_(-1),
        isup_(-1), expiry_(-1.0), next_(NULL) {}
    LinkState(nsaddr_t src, nsaddr_t dest, int seq, int isup, Time expiry) {
        src_ = src;
          dest_ = dest;
          seq_ = seq;
          isup_ = isup;
          expiry_ = expiry;
        next_ = NULL;
    }
};
```

---

#### 3.1.2 IARPAgent类内成员PeripheralNodeList类

 PeripheralNodeList类及其内部的PeripheralNode类

```cpp
//维护单个边界节点的类
class PeripheralNode
{
  public:
    // Data...
    nsaddr_t addr_;        // 节点的地址
    int coveredFlag_;    // Only For IERP
    PeripheralNode *next_;    // 指向下一个元素的指针

    // 构造方法...    
    PeripheralNode() : addr_(-1), coveredFlag_(FALSE), next_(NULL) {}
    PeripheralNode(nsaddr_t addr, int coveredFlag) : addr_(addr),
        coveredFlag_(coveredFlag), next_(NULL) {}
};
//维护边界节点信息的链表
class PeripheralNodeList
{
  public:
    // Data...
    PeripheralNode *head_;//头
    int numPerNodes_;    //内部元素数量

    // 构造器...    
    PeripheralNodeList() : head_(NULL), numPerNodes_(0) {}

    // 方法...
    void addPerNode(nsaddr_t addr, int coveredFlag);
    int findPerNode(nsaddr_t addr);
    void copyList(PeripheralNodeList *newList);
    void freeList();
};
```

---

#### 3.1.3 IARPAgent类内成员InnerRouteList 类

```cpp
//区域内单个路径表
class InnerRoute
{
  public:
    // Data...
    nsaddr_t addr_;        // Destination Address
    nsaddr_t nextHop_;    // 接近destination的下一跳地址
    int numHops_;        // # of hops to the Destination 到目的地已经过了多少跳
    InnerRoute *next_;    // Next entry in the Routelist 路径表指向下一个元素
    InnerRoute *predecessor_; //构建 bordercast tree 边界广播

    // 构造器...
    InnerRoute() : addr_(-1), nextHop_(-1), numHops_(-1), next_(NULL),
        predecessor_(NULL) {}
    InnerRoute(nsaddr_t addr, nsaddr_t nextHop, InnerRoute *predecessor, int numHops) {
        addr_ = addr;
        nextHop_ = nextHop;
        predecessor_ = predecessor;
        numHops_ = numHops;
        next_ = NULL;
    }
};
//区域内维护路径表的链表
class InnerRouteList
{
  private:    
    InnerRoute *tail_;    // Used in addRoute()... 
  public:    
    // Data...
    InnerRoute *head_;
    int numRoutes_;

    // 构造器...
    InnerRouteList() : tail_(NULL), head_(NULL), numRoutes_(0) {}

    // 方法...
    void addRoute(nsaddr_t addr, nsaddr_t nextHop, InnerRoute *predecessor,
        int numHops);
    int findRoute(nsaddr_t addr, InnerRoute **handle);
    void freeList();
};
```

----

##### 3.1.3.1 InnerRouteList类的addRoute方法的实现

```cpp
/* 1. 创造一个Route并将其加入到链表尾部 */
void 
InnerRouteList::addRoute(nsaddr_t addr, nsaddr_t nextHop, InnerRoute *predecessor, int numHops) {
    InnerRoute *newRoute = new InnerRoute(addr, nextHop, predecessor, numHops);
    if(newRoute == NULL) {    // Check for Allocation Error
        printf("### Memory Allocation Error in [InnerRouteList::addRoute] ###");
        exit(0);
    }
    //如果为空链表
    if(head_==NULL && tail_==NULL) {// Empty list
        head_ = newRoute;
        tail_ = newRoute;
    } else {            // Normal Case
        tail_->next_ = newRoute;
        tail_ = newRoute;
    }
    numRoutes_++;            // Increment # of routes
}
```

---

#### 3.1.4 IARPAgent类内成员IARPUpdateDetectedList类

```cpp
//维护单个IARP更新信息
class IARPUpdate
{
  public:
    // Data...
    nsaddr_t updateSrc_;    // Address of the sender（发送方的地址）
    int seq_;            // Sequence Number
    Time expiry_;        // Expiry Time       （超时时间）
    IARPUpdate *next_;    // Pointer to the Next Element

    // 构造器...
    IARPUpdate() :     updateSrc_(-1), seq_(-1), expiry_(-1), next_(NULL) {}
    IARPUpdate(nsaddr_t updateSrc, int seq, Time expiry) {
        updateSrc_ = updateSrc;
        seq_ = seq;
        expiry_ = expiry;
        next_ = NULL;
    }
};
//维护更新的链表
class IARPUpdateDetectedList
{
  public:
    // Data...
    IARPUpdate *head_;    //链表头
    int numUpdates_;    //链表内元素数量

    // 构造器...
    IARPUpdateDetectedList() : head_(NULL), numUpdates_(0) {}

    // 方法...
    void addUpdate(nsaddr_t updateSrc, int seq, Time expiry);
    int findUpdate(nsaddr_t updateSrc, int seq);
    void purgeExpiredUpdates();
    void freeList();
};
```

---

##### 3.1.4.1 IARPUpdateDetectedList类内方法purgeExpiredUpdates()

```cpp
/* 3. Remove all Updates for which the lifetime is expired. */
//移除掉所有超时的更新信息
void
IARPUpdateDetectedList::purgeExpiredUpdates() {
    //如果链表为空
    if(numUpdates_ == 0) {
        return;        // Nothing to do
    }

    // [case-1]: Leaving the 1st case(head_ case)
    //从head->next开始判断
    IARPUpdate *prev, *cur;
    prev = head_;
    cur = head_->next_;
      Time now = Scheduler::instance().clock();     //获取当前时间
    for(int i=1; cur!=NULL; i++) {
        //如果超时
        if(cur->expiry_<now) { // Delete the Update
            prev->next_ = cur->next_;
            IARPUpdate *toBeDeleted = cur;
            cur = cur->next_;
            delete toBeDeleted;
            numUpdates_--;        // Decrement Number of Links
        } else {
            prev = cur;        // Advance the Pointers
            cur = cur->next_;                
        }
    }
    // [case-2]: head_ case...
    //再单独判断head
    if(head_!=NULL) {
        if(head_->expiry_<now) {
            IARPUpdate *toBeDeleted = head_;
            head_ = head_->next_;
            delete toBeDeleted;
            numUpdates_--;
        }
    }    
}
```

---

#### 3.1.5 IARPAgent类内成员IARPPeriodicUpdateTimer类

该类规划IARP 更新频度

```cpp
/* [SUB-SECTION-3.5]------------------------------------------------------------
 * PERIODIC-UPDATE TIMER:- PERIODIC UPDATE XMISSION
 * -----------------------------------------------------------------------------
 */
class IARPPeriodicUpdateTimer : public Handler
{
 public:
    // Data...
    ZRPAgent *agent_;        // 指向ZRP-Agent
    Event intr_;    
    int min_iarp_update_period_;    // Min-Update Interval 最小更新时间
    int max_iarp_update_period_;    // Max-Update Interval 最大更新时间
    Time lastUpdateSent_;        // Last-Update Sent Time 上一次更新时间

    // Constructor...
    IARPPeriodicUpdateTimer(ZRPAgent* agent) : agent_(agent),
        min_iarp_update_period_(DEFAULT_MIN_IARP_UPDATE_PERIOD),
        max_iarp_update_period_(DEFAULT_MAX_IARP_UPDATE_PERIOD),
        lastUpdateSent_(0.0) {}

    // Methods...
      void handle(Event*);          // function handling the event
      void start(double thistime);
};
```

---

##### 3.1.5.1 IARPPeriodicUpdateTimer类内方法start()

```cpp
/* 
* 开启timer
*/
void 
IARPPeriodicUpdateTimer::start(double thistime) {
    Scheduler::instance().schedule(this, &intr_, thistime);
}
```

---

##### 3.1.5.2  IARPPeriodicUpdateTimer类内方法handle()

```cpp
/* 2. Each time it sends IARP update IF NECESSARY(based on flag value). */
// 判断是否需要发送IARP update
void 
IARPPeriodicUpdateTimer::handle(Event* e) {
    // [Check]: 如果当前半径为1 则取消发送 并规划下一次发送
    if(agent_->radius_ == 1) {
        // Schedule next update in min_iarp_update_period sec
        Scheduler::instance().schedule(this, &intr_, min_iarp_update_period_);    // Unnecessary but for adaptive-case
        return;
    }

    // [Task-1]: 检查更新是否是必要的
    // [Condition: (邻居表没有变化) AND (max_update_time is not elapsed)]
    Time now = Scheduler::instance().clock();     // get the time
    if((agent_->iarpAgt_).updateSendFlag_==FALSE && (now-lastUpdateSent_)<max_iarp_update_period_) {
                    if(DEBUG) { // [Log: MISS_IARP_UPDATE]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | MISS_IARP_UPDATE | No Neighbor Changes | -Tlsu %d | ",
                    agent_->myaddr_, now, min_iarp_update_period_);
                    agent_->print_tables(); printf("\n"); 
                    } // [Log: End]
        // schedule next update in min_iarp_update_period sec
        Scheduler::instance().schedule(this, &intr_, min_iarp_update_period_);
        return;        
    }

    // [Task-2]:发送更新
    //判断邻居节点是否为空
    if((agent_->ndpAgt_).neighborLst_.isEmpty() == FALSE) {    // IF neighbor-list is NOT empty
        Packet* p;        // 创建传送的包的指针临时变量
        // [SubTask-1]: 创建包 (TTL should be 'Radius-1')
        // 包类型为IARP_UPDATE,IP地址为广播，跳数为当前半径-1
          p = (agent_->pktUtil_).pkt_create(IARP_UPDATE, IP_BROADCAST, agent_->radius_-1); 
        // 在包中创建链路状态的空间
        (agent_->pktUtil_).pkt_add_LSU_space(p, (agent_->ndpAgt_).neighborLst_.numNeighbors_);
        //获取包内的zrp包头的地址
        hdr_zrp *hdrz = HDR_ZRP(p);
        //加入邻居节点的信息
        // [SubTask-2]: Add Neighbor-Info(LSUs)
        Neighbor *cur = (agent_->ndpAgt_).neighborLst_.head_;    
        for(int i=0; i<(agent_->ndpAgt_).neighborLst_.numNeighbors_; i++) {
            hdrz->links_[i].src_ = agent_->myaddr_;     // My Address
            hdrz->links_[i].dest_ = cur->addr_;        // Neighbor's Address
            hdrz->links_[i].isUp_ = cur->linkStatus_;    // Neighbor's Link-Status
            cur = cur->next_;
        }
        //更新包头内链路状态的数量
        hdrz->numlinks_ = (agent_->ndpAgt_).neighborLst_.numNeighbors_;
        // [SubTask-3]: 广播包
        (agent_->pktUtil_).pkt_broadcast(p, 0.00);
                    if(DEBUG) { // [Log: XMIT_IARP_UPDATE]
                    Time now = Scheduler::instance().clock(); // get the time
                    hdr_zrp *hdrz = HDR_ZRP(p);           // access ZRP part of pkt header
                      hdr_ip *hdrip = HDR_IP(p);
                    printf("\n_%d_ [%6.6f] | XMIT_IARP_UPDATE | -S %d -IS %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrip->saddr(), hdrz->seq_);    
                    (agent_->pktUtil_).pkt_print_links(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
        // [SubTask-4]: IARP State Change... [For Control Flooding]
        // 加入Detected-Update Cache
        (agent_->iarpAgt_).upLst_.addUpdate(hdrz->src_, hdrz->seq_, 
                    now+(agent_->iarpAgt_).updateLifeTime_); // Add into Detected-Update Cache
        //并将更新传送flag设置为false
        (agent_->iarpAgt_).updateSendFlag_ = FALSE;    // Make the flag [FALSE]
        //上一次更新设置为当前时间
        lastUpdateSent_ = now;                // Store Last_Update_Send Time
        // [SubTask-5]: Drop DOWN Neighbors... [For NDP, removing DOWN Neighbors]
        //移除掉LINKDOWN的邻居节点
        (agent_->ndpAgt_).neighborLst_.purgeDownNeighbors();    // Purge all DOWN Neighbors from NeighborList
    } else {    // neighbor-list is EMPTY
                    if(DEBUG) { // [Log: IARP_ISOLATED_NODE ]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | IARP_ISOLATED_NODE | No Neighbors | ",
                    agent_->myaddr_, now);
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
    }    

    // [Task-3]: Schedule next update in min_iarp_update_period sec
    Scheduler::instance().schedule(this, &intr_, min_iarp_update_period_);
}
```

---

#### 3.1.6  IARPAgent类内成员IARPExpirationTimer类

该类为过期检查定时器：- 用于链路状态和路由的过期

```cpp
/* [SUB-SECTION-3.6]------------------------------------------------------------
 * EXPIRATION-CHECK TIMER:- FOR EXPIRATION OF LINKSTATE AND ROUTES
 * -----------------------------------------------------------------------------
 */
class IARPExpirationTimer : public Handler
{
  public:    
    // Data...
    ZRPAgent *agent_;        // 指向ZRP-Agent的指针
    Event intr_;
    int expiration_check_period_;    // Expiration-Check-Interval

    // Constructor...
    IARPExpirationTimer(ZRPAgent* agent) : agent_(agent),
        expiration_check_period_(DEFAULT_EXPIRATION_CHECK_PERIOD) {}

    // Methods...
      void handle(Event*);          // function handling the event
      void start(double thistime);
};
```

---

##### 3.1.6.1 IARPExpirationTimer类的Start()方法

```cpp
/* 1. Starts the Timer. */
void 
IARPExpirationTimer::start(double thistime) {
     Scheduler::instance().schedule(this, &intr_, thistime);
}
```

---

##### 3.1.6.2 IARPExpirationTimer类的handle()方法

```cpp
/* 2. Removes the EXPIRED link-state entries & EXPIRED detected Updates periodically. */
//定期删除过期的link-state entries
//定期删除过期的更新信息
void 
IARPExpirationTimer::handle(Event* e) {
    // [Task-1]: 
    // Purge all expired Links from the Link-State List...
    // 从链接状态列表中清除所有过期链接...
    // & Purge all expired detected Updates from IARP Update List...
    // 从 IARP 更新列表中清除所有检测到的过期更新.    
    int numPurgedLinks=0; 
    numPurgedLinks = (agent_->iarpAgt_).lsLst_.purgeLinks();
    if (numPurgedLinks > 0) {
        (agent_->iarpAgt_).buildRoutingTable();      // Rebuild Routing Table
    }
    (agent_->iarpAgt_).upLst_.purgeExpiredUpdates();

    // [Task-2]: schedule next expiration event
    Scheduler::instance().schedule(this, &intr_, expiration_check_period_ );    
}
```

---

#### 3.1.7 IARPAgent类内方法startUp()

```cpp
/* 1. Starting Timers & Clearing Lists(Tables). */
// 开启Timer
// 清空链表
void 
IARPAgent::startUp() {
    // [Task-1]: Start Timers...
    double startUpJitter;

    PeriodicUpdateTimer_.start(startUpJitter);

    startUpJitter =  Random::uniform(startup_jitter_);
    ExpirationTimer_.start(startUpJitter);

    // [Task-2]: Clear all Lists...
    lsLst_.freeList();    // Link State List
    pnLst_.freeList();    // Peripheral Node List
    irLst_.freeList();    // Inner Route List
    upLst_.freeList();    // Detected IARP_Update List
}
```

---

#### 3.1.8 IARPAgent类内方法buildRoutingTable()

不是很理解这部分，需要之后重新检查

```cpp
/* 2. Create Inner Route List based on Link State List. */
// 根据当前的链路状态去更新内部路由表
void 
IARPAgent::buildRoutingTable() {
    // [Task-1]: Clear the existing Inner Route List & Peirpheral Node List...
    // 清空已经存在的内部路由表
    // 清空已经存在的外围节点表
    irLst_.freeList();
    pnLst_.freeList();

    // [Task-2]: Purge the Expired links & DOWN links...
    // 清除过期的链路状态
    lsLst_.purgeLinks();

    // [Task-3]: Generate BFS tree from existing Link-State list & store them into Inner Route List...
    // 从现有链路状态列表生成 BFS 树并将它们存储到内部路由列表中...
    // 在innerList里加入初始节点,生成头，头节点为自己的地址 跳数为0
    irLst_.addRoute(agent_->myaddr_, agent_->myaddr_, NULL, 0);    // Adding First Entry [0 is IMPortant]
    InnerRoute *curNode = irLst_.head_;    // We just added first Entry
    int numHops = 0;    // Intializing...[0 is IMPortant]
    do {
        // Scan the Link-State list...
        // 扫描当前Link-State链表
        // 获取当前的链路头
        LinkState *curLink=lsLst_.head_;
        InnerRoute *dummyHandle;// Needed for 'irLst_.findRoute()'...
        int flagFoundNewDest;
        //32bit地址 下一跳
        nsaddr_t newFoundDest, nextHop;
        //遍历链路状态表
        for(int i=0; i<lsLst_.numLinks_; i++) {
            //初始化数据
            flagFoundNewDest=FALSE;
            newFoundDest=-1;
            nextHop=-1;    
            // [SubTask-1]: Find New Destination...
            if(curNode->addr_==curLink->src_) {        // Found New Destination [Case-1]    
                newFoundDest = curLink->dest_;
                flagFoundNewDest = TRUE;    
            } else if(curNode->addr_==curLink->dest_) {    // Found New Destination [Case-2]
                newFoundDest = curLink->src_;
                flagFoundNewDest = TRUE;
            }
            // [SubTask-2]: Check if this destination has been detected before...
            // 检查之前是否检测到此目的地...
            // If [NO] then add new route entry...    
            // 如果不是 意味着新的路径节点
            if(flagFoundNewDest==TRUE && irLst_.findRoute(newFoundDest, &dummyHandle)==FALSE) {
                // Find numHops for Found Destination... [Here is WHY- Source's 0 value is IMPortant]    
                numHops = curNode->numHops_ + 1;
                // Find Next Hop for Found Destination...[2 cases-Neighbors and others]
                if(numHops == 1) {
                    nextHop = newFoundDest;        // [Dest = Neighbor] So, [Nexthop = Neighbor]
                } else if (numHops > 1) {
                    nextHop = curNode->nextHop_;    // [Dest = Others] So, [CurNode's route]    
                }
                // Routes are added at the Tail in the list...
                irLst_.addRoute(newFoundDest, nextHop, curNode, numHops);
                // Check if it is a peripheral Node
                if(numHops == agent_->radius_) {    // It's also a peripheral Node
                    if(pnLst_.findPerNode(newFoundDest) == FALSE) {
                        pnLst_.addPerNode(newFoundDest, FALSE);    // Keep it 'coveredFlag=FALSE' for IERP...
                    }
                }
            }
            curLink = curLink->next_;
        }
        curNode = curNode->next_;
    } while(curNode!=NULL);
}
```

---

#### 3.1.9 IARPAgent类内方法addRouteInPacket()

```cpp
/* 3. Add Existing route in the packet. */
// 在数据包中添加现有的路由
void 
IARPAgent::addRouteInPacket(nsaddr_t dest, Packet *p) {
    //获取包的zrp包头指针
    hdr_zrp *hdrz = HDR_ZRP(p);
    //定义临时存储InnerRoute的变量
    InnerRoute *handleToIARPRoute = NULL;
    int foundIARPRoute, routeLen;
    //在innerRoutelist查找的flag
    //在他的路由半径内查找
    foundIARPRoute = irLst_.findRoute(dest, &handleToIARPRoute);
    assert(foundIARPRoute == TRUE);    // Route Must be Found;
    // 如果路径被发现
    if(foundIARPRoute == TRUE) {
        //将路径长度+1
        routeLen = handleToIARPRoute->numHops_+1;        // adding 1 for Source entry...
        //在包内创造route的空间
        (agent_->pktUtil_).pkt_add_ROUTE_space(p, routeLen);    // Allocate memory to store route
        // 按照BFS添加内部路由
        InnerRoute *curNode = handleToIARPRoute;
        for(int i=routeLen-1; i>=0; i--) {    // Copy route
            hdrz->route_[i] = curNode->addr_;    
            curNode = curNode->predecessor_;
        }
        //确认包头的route[0]为自己
        //确认包头的route[len-1]为目标节点
        assert(hdrz->route_[0] == agent_->myaddr_); 
        assert(hdrz->route_[routeLen-1] == dest);    
        //也设置route的长度
        hdrz->routelength_ = routeLen;    // Also Route-length is set.
    }
}
```

---

#### 3.1.9 IARPAgent类内方法recv_IARP_UPDATE(Packet* p)

该方法为接收到IARP_UPDATE包下的调用

```cpp
/* 4. 接收到 IARP Updates. */
void 
IARPAgent::recv_IARP_UPDATE(Packet* p) {
                    if(DEBUG) { // [Log: RECV_IARP_UPDATE]
                    Time now = Scheduler::instance().clock(); // get the time
                    hdr_zrp *hdrz = HDR_ZRP(p);           // access ZRP part of pkt header
                    hdr_ip *hdrip = HDR_IP(p);
                    printf("\n_%d_ [%6.6f] | RECV_IARP_UPDATE | -S %d -IS %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrip->saddr(), hdrz->seq_);    
                    (agent_->pktUtil_).pkt_print_links(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    //获取包的zrp包头指针地址
    hdr_zrp *hdrz = HDR_ZRP(p);
    // 检查其是否已经被检测到
    // 当检测到了 就丢弃该包(该upLst的更新生命周期取决于DEFAULT_UPDATE_LIFETIME)
    // [Task-1]: Check if ??this update has been already detected??
    if(upLst_.findUpdate(hdrz->src_, hdrz->seq_) == TRUE) {
        (agent_->pktUtil_).pkt_drop(p);// Do Nothing and Drop the Packet
        return;  // Return back
    }
    // 对于数据包中的所有 LSU，更新链路状态列表
    // [Task-2]: For all LSUs in the packet, Update the Link-State List...
    Time now = Scheduler::instance().clock(); // 获取当前的时间
    LinkState *handleToFoundEntry = NULL;
    int changeLSLstFlag = FALSE;
    for(int i=0; i<hdrz->numlinks_; i++) {
        // 如果当前LSU已经存在，那么更新他的过期时间和链路状态
        // [Case-1]: If [current LSU is present] then [Just update its Expiry time & Link-State]
        if(lsLst_.findLink(hdrz->links_[i].src_, hdrz->links_[i].dest_, &handleToFoundEntry) == TRUE) {
            if(hdrz->links_[i].isUp_ == LINKDOWN) {
                changeLSLstFlag = TRUE;        // One link is DOWN, Must update the Routing-Table
                handleToFoundEntry->isup_ = hdrz->links_[i].isUp_;// Update Attributes - Link-State
            } else {
                handleToFoundEntry->isup_ = hdrz->links_[i].isUp_;// Update Attributes - Link-State
                handleToFoundEntry->expiry_ = now+linkLifeTime_;  // Update Attributes - Expiry Time
            }
        // 如果当前链路状况不存在
        // [Case-2]: If [current LSU is not present] then [Add LSU]    
        } else {
            lsLst_.addLink(hdrz->links_[i].src_, hdrz->links_[i].dest_, hdrz->seq_, hdrz->links_[i].isUp_,
            now+linkLifeTime_);
            changeLSLstFlag = TRUE;    // At least One new link is added...
        }
    }

    // 如果检测到任何链路状况变更
    // [Task-3]: If any Change is detected in Link-State List, Then Build the Routing Table...
    if(changeLSLstFlag == TRUE) {
        //先更新内部路由表
        buildRoutingTable();
        //检查有什么合适的目的地可以发送
        agent_->route_SendBuffer_pkt();    // Send Buffered packets...
    }

    // 将其加入到更新的链表，之后查看该链表决定IARP是否更新
    // [Task-4]: Add this in the Detected IARP_UPDATE list...
    upLst_.addUpdate(hdrz->src_, hdrz->seq_, now+updateLifeTime_);

    //泛洪，将其提供给下一个节点，只要TIME TO LIVE没有为0
    // [Task-5]: Forward the Same Update packet if ttl is not zero.
    hdr_ip *hdrip = HDR_IP(p);
    hdr_cmn *hdrc = HDR_CMN(p);
    int ttl = hdrip->ttl() - 1;  // Decrement the TTL value
    //先减TTL，再判断
    if(ttl <= 0) {//TTL==0 丢弃
        (agent_->pktUtil_).pkt_drop(p); 
    } else {
        hdrc->direction() = hdr_cmn::DOWN;    // Set Fields
        //重新设置广播src_ip
        hdrip->saddr() = agent_->myaddr_;    // Re-broadcaster
        //设置TIME TO LIVE
        hdrip->ttl() = ttl;            // Decremented TTL value
        //再次泛洪,设置泛洪的flag=1
        hdrz->forwarded_ = 1;         // Setting Forwarding field
        (agent_->pktUtil_).pkt_broadcast(p, 0.00); // broadcast pkt
                    if(DEBUG) { // [Log: XMIT_IARP_UPDATE]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IARP_UPDATE | -S %d -IS %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrip->saddr(), hdrz->seq_);    
                    (agent_->pktUtil_).pkt_print_links(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    }
}
```

---

#### 3.1.10 IARPAgent类内方法recv_IARP_DATA(Packet* p)

该方法接受IARP_DATA

```cpp
/* 5. Receiving IARP DATA(Local Traffic) [Either I need to route the pkt OR I am the DESTINATION]. */
// 接受IARP DATA
void 
IARPAgent::recv_IARP_DATA(Packet* p) {
      //获取IP
      hdr_ip *hdrip = HDR_IP(p);
      //获取普通包头
      hdr_cmn *hdrc = HDR_CMN(p);
      //获取zrp包头
      hdr_zrp *hdrz = HDR_ZRP(p);
                    if(DEBUG) { // [Log: RECV_IARP_DATA]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_IARP_DATA | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->seq_);    
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]    
    //如果我是目标节点
    // [Case-1]: If I am the Destination...
    if(hdrz->dest_ == agent_->myaddr_) {
                    if(DEBUG) { // [Log: RECV_CBR_DATA]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_CBR_DATA | -S %d -D %d | -SEQ %d -Type IARP -Delay %6.6f | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrz->seq_, now-hdrz->pktsent_);
                    (agent_->pktUtil_).pkt_print_route(p); printf("\n");
                    } // [Log: End]
        // Delete the route info from the packet [Not Needed Anymore]
        // (agent_->pktUtil_).pkt_free_ROUTE_space(p);// [Currently creates memory problems-Not implemented]
        // copy ecnapsulated data back to this packet
        hdrc->ptype() = hdrz->enc_ptype_;
        hdrc->direction() = hdr_cmn::UP;     // and is sent up
        hdrc->addr_type_ = NS_AF_NONE;
        hdrc->size() -= IP_HDR_LEN;          // cut the header
        hdrip->dport() = hdrz->enc_dport_;
        hdrip->saddr() = hdrz->src_;
        hdrip->daddr() = hdrz->dest_;
        hdrip->ttl() = 1;
        (agent_->pktUtil_).pkt_send(p, agent_->myaddr_, 0.00);
    //　我不是目标节点，泛洪该数据包
    // [Case-2]: Forward the packet to Next Hop...
    } else {
        hdrz->routeindex_++;    // Move 1 hop (FORWARD) into the route(Should be My address)
        assert(hdrz->route_[hdrz->routeindex_] == agent_->myaddr_); assert(hdrz->routeindex_+1 < hdrz->routelength_);
        hdrc->direction() = hdr_cmn::DOWN;
        hdrip->saddr() = agent_->myaddr_;
        hdrip->daddr() = hdrz->route_[hdrz->routeindex_+1];
        (agent_->pktUtil_).pkt_send(p, hdrip->daddr(), 0.00);
                    if(DEBUG) { // [Log: XMIT_IARP_DATA]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IARP_DATA | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->seq_);    
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    }
}
```

---

#### 3.1.11 IARPAgent类内方法print_tables()

```cpp
/* 6. Print all Tables. */
// 打印所有的表
void 
IARPAgent::print_tables() {
    // [Task-1]: Print current Peripheral nodes
    printf("Peripheral Nodes:[ ");
    if(pnLst_.numPerNodes_ == 0) {
        printf("EMPTY");
    }
    PeripheralNode *curPN = pnLst_.head_;
    for(int i=0; i<pnLst_.numPerNodes_; i++) {
        printf("%d ", curPN->addr_);
        curPN = curPN->next_;
    }
    printf("]");

    // [Task-2]: Print All Local Routes
    printf("Local Routes:[ ");
    if(irLst_.numRoutes_ == 0){
        printf("EMPTY");
    }
    InnerRoute *cur = irLst_.head_;
    for(int i=0; i<irLst_.numRoutes_; i++) {
        printf("(%d:%d), ", cur->addr_, cur->nextHop_);
        cur = cur->next_;
    }
    printf("]");
}
```

---

## 4.IERP (被动协议)

---

### 4.1 IERPAgent类

该类为ZRP的被动协议控制类

```cpp
/* [SUB-SECTION-4.5]------------------------------------------------------------
 * IERP AGENT:- INTER-ZONE ROUTING AGENT
 * -----------------------------------------------------------------------------
 */
class IERPAgent
{
  //私有方法
  private:    
    void addMyAddressToRoute(Packet *pold, Packet *pnew);
    void markCoveredPN(DetectedQuery *queryInCache, nsaddr_t lastBC);
  //公共方法
  public:
    // Data...
    ZRPAgent *agent_;      //指向ZRPAgent的指针
    int brpXmitPolicy_;    //边界广播flag(0: UNICAST, 1: MULTICAST).

    // Lists...
    DetectedQueryList dqLst_; //检查到查询的表
    SentQueryList sqLst_;    //它保留所有已发送的路由请求。
    int queryLifeTime_;        // DEFAULT_QUERY_LIFETIME  默认询问生存时间
    int routeLifeTime_;        // DEFAULT_ROUTE_LIFETIME  默认路由生存时间
    int queryRetryTime_;    // DEFAULT_QUERY_RETRY_TIME 默认询问再发时间 

    // Timers...
    //过期检查计时器：- 用于 IERP 请求的过期
    IERPExpirationTimer ExpirationTimer_; 
    int startup_jitter_;    // For starting the Timers

    // 构造器...
    IERPAgent(ZRPAgent *agent) : agent_(agent), 
        brpXmitPolicy_(DEFAULT_BRP_XMIT_POLICY), 
        queryLifeTime_(DEFAULT_QUERY_LIFETIME), 
        routeLifeTime_(DEFAULT_ROUTE_LIFETIME),     
        queryRetryTime_(DEFAULT_QUERY_RETRY_TIME),
        ExpirationTimer_(agent), 
        startup_jitter_(DEFAULT_STARTUP_JITTER) {}

    // Methods...
    void startUp();
    void recv_IERP_ROUTE_REQUEST_UNI(Packet* p);
    void recv_IERP_ROUTE_REQUEST_MC(Packet* p);
    void recv_IERP_ROUTE_REPLY(Packet* p);
    void recv_IERP_ROUTE_ERROR(Packet* p);
    void recv_IERP_DATA(Packet* p);
    int addLinkStateFromRoute(nsaddr_t *route, int size);
    int removeLinkStateFromBrokenRoute(nsaddr_t lnkSrc, nsaddr_t lnkDest);
    void print_tables();
};
```

---

#### 4.1.1 IERPAgent类内私有方法addMyAddressToRoute()

```cpp
/* 2. Appends own address to the route. */
//将自己的地址附加到路由
void 
IERPAgent::addMyAddressToRoute(Packet *pold, Packet *pnew) {
    //旧的zrp的包头的指针
    hdr_zrp *hdrzold = HDR_ZRP(pold);
    //新的zrp的包头的指针
    hdr_zrp *hdrznew = HDR_ZRP(pnew);
    //路由长度的临时变量
    int routeLen;
    //设置新的路有长度的变量
    routeLen = hdrzold->routelength_ + 1;    // For My Address
    // [Task-1]: Allocate memory to store route
    // 创建内部的存储空间 大小为routeLen
    (agent_->pktUtil_).pkt_add_ROUTE_space(pnew, routeLen);    
    // [Task-2]: Copy 1st part of route
    //将旧的路由线路拷贝到新的空间
    for(int i=0; i<hdrzold->routelength_; i++) {        
        hdrznew->route_[i] = hdrzold->route_[i];    // ['0' to 'hdrzold->routelength_ - 1']
    }
    // 加入自己的地址
    // [Task-3]: Copy 2nd part of route
    hdrznew->route_[routeLen-1] = agent_->myaddr_;
    //修改路有的长度
    // [Task-4]: Update Route-Length
    hdrznew->routelength_ = routeLen;    // Also Route-length is set.
}
```

---

#### 4.1.2 IERPAgent类内私有方法markCoveredPN()

不是很明白该方法

```cpp
/* 3. Marks all covered peripheral nodes based on Last bordercaster Info. */
// 根据 Last bordercaster Info 标记所有覆盖的外围节点。
void 
IERPAgent::markCoveredPN(DetectedQuery *queryInCache, nsaddr_t lastBC) {
    //周边节点的存储指针
    PeripheralNode *cur = NULL;
    // 将其指向检测查询缓存的外围节点的head
    cur = (queryInCache->pnLst_).head_;
    // InnerRoute的临时变量
    InnerRoute *handleToFoundRoute = NULL;
    // flag
    int foundRouteFlag;
    //
    for(int i=0; i<(queryInCache->pnLst_).numPerNodes_; i++) {
        foundRouteFlag = (agent_->iarpAgt_).irLst_.findRoute(cur->addr_, &handleToFoundRoute);
        if(foundRouteFlag == TRUE) {    // Route to Peripheral node exists [It must exist]
            if(handleToFoundRoute->nextHop_ == lastBC) {
                cur->coveredFlag_ = TRUE;    // Mark this Node
            }
        }
        cur = cur->next_;    // Advance the Pointer
    }
}
```

---

#### 4.1.3 IERPAgent类内成员DetectedQueryList类

```cpp
//检测到查询的单个存储类
class DetectedQuery
{
  public:
    // Data...
    nsaddr_t src_;        // 32 bits 源节点
    nsaddr_t dest_;        // 32 bits 目标节点
    int queryID_;        // Query ID 
    Time expiry_;        // Expiry of this detected Entry 超时时间
    PeripheralNodeList pnLst_;    // Query-Coverage Info 查询覆盖信息
    //是否需要发送 或者 泛洪该查询信息的flag
    int querySentFlag_;    // Whether the query is Sent/Forwarded or Not
    int totalReplySent_;  //已为此查询发送了多少答复。
    DetectedQuery *next_;     // 指向下一个Query

    // 构造方法...
    DetectedQuery() : src_(-1), dest_(-1), queryID_(-1), expiry_(-1.0),
        querySentFlag_(0), totalReplySent_(0), next_(NULL) {}
    DetectedQuery(nsaddr_t src, nsaddr_t dest, int queryID, Time expiry,
        PeripheralNodeList *curPNLst) {
          src_ = src;
        dest_ = dest;
        queryID_ = queryID;
        expiry_ = expiry;
        querySentFlag_ = FALSE;
        next_ = NULL;
        totalReplySent_ = 0; 
        // Add peripheral node list...
        //添加外围节点表
        if(curPNLst == NULL) {// UNICAST CASE。单播
            //不做任何事情，不需要使用到覆盖节点的信息
            // Do Nothing [We wont use coverage info]...
        } else {        // MULTICAST CASE 广播
            // Copies the peripheral-node-list to 'pnLst'
            // 拷贝当前节点的外围节点表
            curPNLst->copyList(&pnLst_);
        }
    }
};

class DetectedQueryList
{
  public:
    // Data...
    DetectedQuery *head_; //查询信息的存储链表的头
    int numQueries_;    //链表内元素

    // 构造器...
    DetectedQueryList() : head_(NULL), numQueries_(0) {}

    // 方法...
    void addQuery(nsaddr_t src, nsaddr_t dest, int queryID, Time expiry,
        PeripheralNodeList *curPNLst);
    int findQuery(nsaddr_t src, nsaddr_t dest, int queryID, DetectedQuery
        **handle);    // Returns TRUE or FALSE.
    void removeQuery(nsaddr_t src, nsaddr_t dest, int queryID);
    void purgeExpiredQueries();
    void freeList();
};
```

---

##### 4.1.3.1 DetectedQueryList类内方法addQuery()

```cpp
/* 1. Add Detected Query at the head. */
//讲问询信息存储到问询链表中
//参数:原节点地址，目标节点地址，问询ID，超时时间，外围节点链表指针
void 
DetectedQueryList::addQuery(nsaddr_t src, nsaddr_t dest, int queryID, Time expiry, PeripheralNodeList *curPNLst) {
    DetectedQuery *newDQ = new DetectedQuery(src, dest, queryID, expiry, curPNLst);    // Create a new Detected Query
    if(newDQ == NULL) {    // Check for Allocation Error
        printf("### Memory Allocation Error in [DetectedQueryList::addQuery] ###");
        exit(0);
    }
    //头插法
    newDQ->next_ = head_;    // Made necessary Joinings            
    head_ = newDQ;
    numQueries_++;        // Increment Number of Links
}
```

##### 4.1.3.2 DetectedQueryList类内方法findQuery()

```cpp
/* 2. Find the Query and 1)send TRUE & the handle OR 2)send FALSE. */
//查找这个问询信息，返回查找结果
//查找这个问询信息，将其交给handle指针
int 
DetectedQueryList::findQuery(nsaddr_t src, nsaddr_t dest, int queryID, DetectedQuery **handle) {
    DetectedQuery *cur;//临时变量
    int foundFlag;//查找flag
    // [i<numQueries_] condition takes care for EMPTY List case...
    cur = head_;
    foundFlag = FALSE;
    //从头查找
    for(int i=0; i<numQueries_; i++) {
        if(cur->src_ == src && cur->dest_ == dest && cur->queryID_ == queryID) {
            foundFlag = TRUE;
            break;
        }
        cur=cur->next_;
    }
    if(foundFlag) {        // If Query is Found
        *handle = cur;    // ...Return the handle of that Query
        return TRUE;    // ...Return Found = TRUE        
    }
    return FALSE;        // Query is NOT found (returning FALSE)
}
```

---

##### 4.1.3.3 DetectedQueryList类内方法removeQuery()

```cpp
/* 3. Find the Query and Delete it. */
//查找到问询并且删除
void 
DetectedQueryList::removeQuery(nsaddr_t src, nsaddr_t dest, int queryID) {
    //临时变量
    DetectedQuery *prev, *cur, *toBeDeleted;
    // 1.头节点需要删除
    // [Case-1]: first Node to Delete...
    if(head_!=NULL) {
        if(head_->src_ == src && head_->dest_ == dest && head_->queryID_ == queryID) {
            toBeDeleted = head_;
            head_ = head_->next_;
            (toBeDeleted->pnLst_).freeList();    // It frees the memory
            delete toBeDeleted;
            numQueries_--;
            return;
        }
    }
    // 2.普通删除
    // [Case-2]: Normal Case...
    prev = head_;
    cur = head_->next_;    
    for(int i=1; i<numQueries_; i++) {
        if(cur->src_ == src && cur->dest_ == dest && cur->queryID_ == queryID) {
            toBeDeleted = cur;
            prev->next_ = cur->next_;    // Make necessary joinings
            (toBeDeleted->pnLst_).freeList();    // It frees the memory
            delete toBeDeleted;
            numQueries_--;
            break;
        }
        prev = cur;
        cur = cur->next_;
    }
}
```

---

##### 4.1.3.3 DetectedQueryList类内方法purgeExpiredQueries()

```cpp
/* 4. Remove all Queries for which the lifetime is expired. */
//删除所有的超出生命周期的问询
void 
DetectedQueryList::purgeExpiredQueries() {
    //1.如果链表为空
    if(numQueries_ == 0){
        return;        // Nothing to do
    }
    // 先抛开头节点
    // [case-1]: Leaving the 1st case(head_ case)
    DetectedQuery *prev, *cur, *toBeDeleted;//临时变量
    prev = head_;
    cur = head_->next_;
    Time now = Scheduler::instance().clock();//获取当前时间
    for(int i=1; cur!=NULL; i++) {
        if(cur->expiry_ < now) { //超时则需要删除该问询
            prev->next_ = cur->next_;//先断开删除节点
            toBeDeleted = cur;
            cur = cur->next_;
            (toBeDeleted->pnLst_).freeList();    // It frees the memory
            delete toBeDeleted;
            numQueries_--;            // Decrement Number of Queries
        } else {
            prev = cur;            //指针前进
            cur = cur->next_;
        }
    }
    //单独判断头节点
    // [case-2]: head_ case...
    if(head_!=NULL) {
        if(head_->expiry_<now) {
            toBeDeleted = head_;
            head_ = head_->next_;        // Make necessary joinings
            (toBeDeleted->pnLst_).freeList();    // It frees the memory
            delete toBeDeleted;
            numQueries_--;            // Decrement Number of Queries
        }
    }
}
```

---

##### 4.1.3.3 DetectedQueryList类内方法freeList()

```cpp
/* 5. Frees the entire list. */
//清空整个链表
void 
DetectedQueryList::freeList() {
    if(numQueries_ == 0) {
        return;
    }

    DetectedQuery *toBeDeleted, *cur;
    cur = head_;
    for(int i=0; i<numQueries_; i++) {
        toBeDeleted = cur;
        cur = cur->next_;
        (toBeDeleted->pnLst_).freeList();    // It frees the memory
        delete toBeDeleted;
    }
    head_ = NULL;
    numQueries_ = 0;
}
```

---

#### 4.1.4 IERPAgent类内成员SentQueryList()

```cpp
/* [SUB-SECTION-4.2]------------------------------------------------------------
 * SENT IERP-REQUEST:- SENT QUERIES CACHE
 * 存储所有已经发送的Queries
 * -----------------------------------------------------------------------------
 */
//单个发送Query的info
class SentQuery
{
  public:
    // Data...
      nsaddr_t src_;      // 32 bits 送信地址 
    nsaddr_t dest_;      // 32 bits 目标地址
    Time expiry_;        // Expiry of this detected Entry 超时时间
    SentQuery *next_;    // Pointer to Next Element 指向下一个的指针

    // Constructors...    
    SentQuery() : src_(-1), dest_(-1), expiry_(-1.0), next_(NULL) {}
    SentQuery(nsaddr_t src, nsaddr_t dest, Time expiry) {
        src_ = src;
        dest_ = dest;
        expiry_ = expiry;
        next_ = NULL;
    }
};
// 存储已发送的Query的链表
class SentQueryList
{
  public:
    // Data...
    SentQuery *head_; //头节点
    int numQueries_; //链表内部数量

    // 构造器...
    SentQueryList() : head_(NULL), numQueries_(0) {}

    // 方法...
    void addQuery(nsaddr_t src, nsaddr_t dest, Time expiry);
    int findQuery(nsaddr_t src, nsaddr_t dest, SentQuery **handle);
    void removeQuery(nsaddr_t src, nsaddr_t dest);
    void purgeExpiredQueries();
    void freeList();
};
```

---

#### 4.1.5 IERPAgent类内成员IERPExpirationTimer

```cpp
/* [SUB-SECTION-4.4]------------------------------------------------------------
 * EXPIRATION-CHECK TIMER:- FOR EXPIRATION OF IERP-REQUESTS
 * -----------------------------------------------------------------------------
 * 它实现了一个计时器，该计时器定期检查 IERP 表中各种条目的到期时间。
 */
 class IERPExpirationTimer : public Handler
{
  public:    
    // Data...
      ZRPAgent *agent_;    // ZRP-Agent指针
      Event intr_;            
    int expiration_check_period_;// 检查时间

    // 构造器...
    IERPExpirationTimer(ZRPAgent* agent) : agent_(agent),
        expiration_check_period_(DEFAULT_EXPIRATION_CHECK_PERIOD) {}

    // 方法...
      void handle(Event*);  // function handling the event
      void start(double thistime);
};
```

---

##### 4.1.5.1 IERPExpirationTimer类内方法start()

```cpp
/* 1. Starts the Timer, Called by startUp() method. */
//该方法开启该定时器
void 
IERPExpirationTimer::start(double thistime) {
    Scheduler::instance().schedule(this, &intr_, thistime);
}
```

---

##### 4.1.5.1 IERPExpirationTimer类内方法handle()

```cpp
/* 2. Purge all expired Queries/Routes. */
// 该方法定时清除所有超市的root或者Query
void
IERPExpirationTimer::handle(Event*) {
    // [Task-1]: Purge all expired Queries from the Query-Detect(/Sent) List...
    //         & Purge all Outer Routes from Outer-Route List...    
    (agent_->ierpAgt_).dqLst_.purgeExpiredQueries();
    (agent_->ierpAgt_).sqLst_.purgeExpiredQueries();
    // 调用zrpAgent内的sendBuf中的清除超时包的方法
    (agent_->sendBuf_).purgeExpiredPackets();    // For ZRP agent
    // [Task-2]: schedule next expiration event
    Scheduler::instance().schedule(this, &intr_, expiration_check_period_ );    
}
```

---

#### 4.1.6 IERPAgent类内方法startUp()

```cpp
//该方法被ZrpAgent::startUp()方法调用
//启动一系列关于IEZRPAgent相关的定时器
void 
IERPAgent::startUp() {
    // [Task-1]: Start Timers...
    double startUpJitter;
      startUpJitter =  Random::uniform(startup_jitter_);
    ExpirationTimer_.start(startUpJitter);

    // [Task-2]: Clear all Lists...
    dqLst_.freeList();    // Detected-Query List
    sqLst_.purgeExpiredQueries();    // Sent-Query List
}
```

---

#### 4.1.7 IERPAgent类内方法 recv_IERP_ROUTE_REQUEST_UNI()

当节点收到路由请求消息时调用它。在收到路由请求时

1.它将查询添加到检测到的查询表中

2.如果节点本身是预期目的地，那么它会创建一个“ROUTE_REPLY”并将其发送回路由请求发起者。

3.如果目的地位于节点的区域中，则它向目的地单播路由请求。

4.节点通过以下步骤对查询进行边界广播

    4.1如果在此之前收到此查询，则什么都不做并返回

    4.2它生成一个邻居集，通过它可以到达所有外围节点。 – 它将查询转发给所有这些邻居。     [多条消息]

不是很了解广播Query的时候的那个NbList如何添加的

```cpp
/* 4. Receiving IERP-route-request. [Unicast Case] */
//接收 IERP 路由请求。 [单播案例]
void 
IERPAgent::recv_IERP_ROUTE_REQUEST_UNI(Packet* p) {
    hdr_zrp *hdrz = HDR_ZRP(p);//获取zrp包头
    hdr_ip *hdrip = HDR_IP(p);//获取IP包头
                    if(DEBUG) { // [Log: RECV_IERP_REQUEST]
                    if(agent_->myaddr_ != hdrz->src_) {    // Only Non-source case...
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_IERP_REQUEST | -S %d -D %d | -IS %d -ID %d | -QID %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");}
                    } // [Log: End]
    // [CommonTask]: Check if this is a new query?? If Yes, Add this Query to the Cache
    // 检查是不是新的Query消息，如果是的话将它加入到DetectedQueryList中
    Time now = Scheduler::instance().clock(); //获取当前时间
    DetectedQuery *handleToFoundDQ = NULL;//临时变量
    int foundQueryFlag;//查询flag
    //先检查DetectedQueryList有没有当前Query
    foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
    if(foundQueryFlag == TRUE) {//查询到该Query在当前链表中
        //如果该Query的地址不是
        if(hdrz->dest_ != agent_->myaddr_) {// Need to send multiple replies if I am dest.
            (agent_->pktUtil_).pkt_drop(p); //丢弃该包
            return;//什么也不做
        }
    } else { // For Control Flooding //否则将其加入DetectedQueryList
        dqLst_.addQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, now+queryLifeTime_, NULL);
    }

    // [Task-1]: If I am the Destination [Create a ROUTE-REPLY(New Packet) and Send it back]]
    // 如果我是目标节点，创建一个ROUTE-REPLY包并且发送他
    if(hdrz->dest_ == agent_->myaddr_) {
        // 检查已发送了多少回复。
        // [SubTask-0]: Check how many replies have been sent.
        DetectedQuery *handleToFoundDQ = NULL;//临时
        int foundQueryFlag;//查找flag
        foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
        assert(foundQueryFlag == TRUE);
        //如果该Query内的最大回复数目超过默认的最大回复数量
        if(handleToFoundDQ->totalReplySent_ >= DEFAULT_MAX_IERP_REPLY) {
            //不需要发送太多回复
            return; // No more replies should be sent.
        } else {
            //先++回复，回复会在下面的代码中创建
            handleToFoundDQ->totalReplySent_++;    // Reply is sent in below code.
        }
        // [SubTask-1]: Create a IERP_REPLY packet ['hdrz->routelength_+1': My address needs to be added]
        //创建一个IERP_REPLY包，内部的路由长度+1，并添加上自己的地址
        Packet *pnew = (agent_->pktUtil_).pkt_create(IERP_REPLY, hdrz->dest_, hdrz->routelength_+1);
        addMyAddressToRoute(p, pnew);    // Add My address to the route
        // [SubTask-2]: Set Parameters...
        // 设定参数
        hdr_zrp *hdrznew = HDR_ZRP(pnew);
        hdrznew->src_ = hdrz->src_;  // Identification fields at Sender of the Query
        hdrznew->dest_ = hdrz->dest_; // Identification fields at Sender of the Query
        hdrznew->seq_ = hdrz->seq_;   // Identification fields at Sender of the Query
        hdrznew->queryID_ = hdrz->queryID_; // Identification fields at Sender of the Query
        hdrznew->routeindex_ = hdrznew->routelength_-1;// My_address in the route
        // [SubTask-3]: Send the Packet
        //发送包
        hdr_ip *hdripnew = HDR_IP(pnew); //读取IP头
        hdripnew->saddr() = agent_->myaddr_;//将IP发送地址设置为自己
        //将目标地址设置为Route的前一个路由
        hdripnew->daddr() = hdrznew->route_[hdrznew->routeindex_-1];// Previous Hop
        (agent_->pktUtil_).pkt_send(pnew, hdripnew->daddr(), 0.00);
                    if(DEBUG) { // [Log: XMIT_IERP_REPLY]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REPLY | -S %d -D %d | -IS %d -ID %d | -QID %d | ",
                    agent_->myaddr_, now, hdrznew->src_, hdrznew->dest_, hdripnew->saddr(),
                    hdripnew->daddr(), hdrznew->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(pnew); agent_->print_tables(); printf("\n");
                    } // [Log: End]
        // 监听回复
        // [SubTask-4]: Snooping the replies.. Adding this info into topology table.
        int routingTableUpdatedFlag=FALSE;
        routingTableUpdatedFlag = addLinkStateFromRoute(hdrznew->route_, hdrznew->routelength_);
        if(routingTableUpdatedFlag == TRUE) {
            agent_->route_SendBuffer_pkt();    // Send Buffered packets...
        }
        // [SubTask-5]: Drop the original Packet 
        (agent_->pktUtil_).pkt_drop(p);    
        return;
    }
    //如果该目标节点在自己的范围内，单播到目标节点
    // [Task-2]: If Destination lies in My Zone.. [Unicast the request to Destination]
    InnerRoute *handleToFoundRoute = NULL;//临时变量
    int foundRouteFlag, alreadySentNb=-1;
    //从内部路径表内查找目标节点，如果查找到则将其给handle
    foundRouteFlag = (agent_->iarpAgt_).irLst_.findRoute(hdrz->dest_, &handleToFoundRoute);
    //如果查找到了区域内节点
    if(foundRouteFlag == TRUE && handleToFoundRoute->nextHop_ != hdrz->lastbc_) {    // Found Route in Zone
        // [SubTask-1]: Create a packet//创建包
        hdr_ip *hdrip = HDR_IP(p);
        int ttl = hdrip->ttl() - 1;    // Reducing the TTL value
        Packet *pnew = (agent_->pktUtil_).pkt_create(IERP_REQUEST, handleToFoundRoute->nextHop_, ttl);
        addMyAddressToRoute(p, pnew);    // Add My address to the route
        // [SubTask-2]: Set Parameters...
        hdr_zrp *hdrznew = HDR_ZRP(pnew);
        hdrznew->src_ = hdrz->src_;    // Identification fields at Sender of the Query
        hdrznew->dest_ = hdrz->dest_;    // Identification fields at Sender of the Query
        hdrznew->seq_ = hdrz->seq_;    // Identification fields at Sender of the Query
        hdrznew->queryID_ = hdrz->queryID_;// Identification fields at Sender of the Query
        hdrznew->lastbc_ = agent_->myaddr_;
        hdrznew->routeindex_ = hdrznew->routelength_ - 1;    // My_address in the route [Just Added b4]
        // [SubTask-3]: Send the Packet...
        hdr_ip *hdripnew = HDR_IP(pnew);
        hdripnew->saddr() = agent_->myaddr_;    
            hdripnew->daddr() = handleToFoundRoute->nextHop_;
        (agent_->pktUtil_).pkt_send(pnew, handleToFoundRoute->nextHop_, 0.00);
        alreadySentNb = handleToFoundRoute->nextHop_;
                    if(DEBUG) { // [Log: XMIT_IERP_REQUEST]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REQUEST | -S %d -D %d | -IS %d -ID %d | -QID %d |",
                    agent_->myaddr_, now, hdrznew->src_, hdrznew->dest_, 
                    hdripnew->saddr(), hdripnew->daddr(), hdrznew->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(pnew); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    }
    //广播该Query
    // [Task-3]: Bordercast The Query...
    // [SubTask-1]: Bordercast it(Find all Next-Hop neighbors to  un-covered-perpheral-nodes)...
    // Bordercast it（找到未覆盖外围节点的所有下一跳邻居）...
    NeighborList NbLst; // Temporary list of query-forwarders
    PeripheralNode *curPN;
    handleToFoundRoute = NULL;
    foundRouteFlag = -1;
    curPN = (agent_->iarpAgt_).pnLst_.head_;// Fetch PN-list from IARP
    //遍历循环查找周边节点链表
    for(int i=0; i<(agent_->iarpAgt_).pnLst_.numPerNodes_; i++) {// For all Peripheral Nodes
        //将周围节点的地址放到内部路由表内查找，并获取查找的Flag
        foundRouteFlag = (agent_->iarpAgt_).irLst_.findRoute(curPN->addr_, &handleToFoundRoute);
        //如果查找到了
        if(foundRouteFlag == TRUE) {    // Add these Neighbors into List
            int foundNbFlag;//定义查找邻居节点的flag
            Neighbor *handleToFoundNb = NULL; // Do Not add Same Neighbors
            //将查找到的内部路由的下一条丢到邻居节点内查询
            foundNbFlag = NbLst.findNeighbor(handleToFoundRoute->nextHop_, &handleToFoundNb);
            if(foundNbFlag == FALSE                     // Not already added
                && handleToFoundRoute->nextHop_ != hdrz->lastbc_     // Not Last BorderCaster
                && handleToFoundRoute->nextHop_ != alreadySentNb    // Not already sent above & No loop
                && (agent_->pktUtil_).pkt_AmIOnTheRoute(p, handleToFoundRoute->nextHop_) == FALSE) {
                // Last-2 args-doesnt matter
                NbLst.addNeighbor(handleToFoundRoute->nextHop_, 0.0, LINKDOWN); 
            }
        }
        curPN = curPN->next_;
    }
    //广播，将查询广播到任何一个在该链表内的邻居节点
    // [SubTask-2]: Bordercast it(Send a Query to every neighbor in the list)...
    if(NbLst.isEmpty() == FALSE) {
        hdr_ip *hdrip = HDR_IP(p);
        int ttl = hdrip->ttl() - 1;// Reducing the TTL value
        if(ttl <= 0) {
            (agent_->pktUtil_).pkt_drop(p);// Drop the Packet [No need to Forward]
            return;
        }
        Neighbor *curNb = NULL;
        curNb = NbLst.head_;
        for(int i=0; i<NbLst.numNeighbors_; i++) {
            // Create a Packet... 
            Packet *pnew = (agent_->pktUtil_).pkt_create(IERP_REQUEST, curNb->addr_, ttl);
            addMyAddressToRoute(p, pnew);    // Add My address to the route
            // Set Parameters...
            hdr_zrp *hdrznew = HDR_ZRP(pnew);
            hdrznew->src_ = hdrz->src_;    // Identification fields at Sender of the Query
            hdrznew->dest_ = hdrz->dest_;    // Identification fields at Sender of the Query
            hdrznew->seq_ = hdrz->seq_;    // Identification fields at Sender of the Query
            hdrznew->queryID_ = hdrz->queryID_;// Identification fields at Sender of the Query
            hdrznew->lastbc_ = agent_->myaddr_;
            hdrznew->routeindex_ = hdrznew->routelength_ - 1;    // My_address in the route [Just Added b4]

            // Send the Packet...
            hdr_ip *hdripnew = HDR_IP(pnew);
            hdripnew->saddr() = agent_->myaddr_;    
                hdripnew->daddr() = curNb->addr_;
            Time jitter;
            jitter =  Random::uniform(IERP_XMIT_JITTER);
            (agent_->pktUtil_).pkt_send(pnew, curNb->addr_, jitter);
                    if(DEBUG) { // [Log: XMIT_IERP_REQUEST]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REQUEST | -S %d -D %d | -IS %d -ID %d | -QID %d |",
                    agent_->myaddr_, now+jitter, hdrznew->src_, hdrznew->dest_, 
                    hdripnew->saddr(), hdripnew->daddr(), hdrznew->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(pnew); agent_->print_tables(); printf("\n");
                    } // [Log: End]
            curNb = curNb->next_;        // Advance the Pointer
        }
        /*[Cumulative Log-Entry]if(DEBUG) { // [Log: EVENT_XMIT_IERP_REQUEST_FORWARDED]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | EVENT_XMIT_IERP_REQUEST_FORWARDED | Node %d has forwarded "
                    "IERP_REQUEST(seq=%d) | [ SRC = %d : DEST = %d ] | To following Neighbors - ",
                    agent_->myaddr_, now, agent_->myaddr_, hdrz->seq_, hdrz->src_, hdrz->dest_);    
                    NbLst.printNeighbors(); (agent_->pktUtil_).pkt_print_route(p); 
                    agent_->print_tables(); printf("\n"); 
                    } // [Log: End]    */
    }
    (agent_->pktUtil_).pkt_drop(p);    // Drop the original Packet 
}
```

---

#### 4.1.8 IERPAgent类内方法recv_IERP_ROUTE_REQUEST_MC()

当节点收到路由请求消息时调用它。在收到路由请求时

1.它将查询添加到检测到的查询表中。

2.如果节点本身是预期目的地，那么它会创建一个“ROUTE_REPLY”并将其发送回路由请求发起者。

3.如果目的地位于节点的区域中，则它向目的地单播路由请求。

4.节点通过以下步骤对查询进行边界广播：

    4.1标记查询覆盖的所有外围节点。

    4.2如果节点不在“MULTICAST_RECEIVER_LIST”中，则什么都不做并返回。

    4.3如果节点在“MULTICAST_RECEIVER_LIST”中，那么，

    4.4它生成一个邻居集，通过它可以到达所有未覆盖的外围节点。

    4.5它将查询转发给所有这些邻居。 [单条消息]

```cpp
/* 5. Receiving IERP-route-request. [Multi-cast Case] */
//接收 IERP 路由请求。 [多播案例]
void 
IERPAgent::recv_IERP_ROUTE_REQUEST_MC(Packet* p) {
    hdr_zrp *hdrz = HDR_ZRP(p);
    hdr_ip *hdrip = HDR_IP(p);
                    if(DEBUG) { // [Log: RECV_IERP_REQUEST]
                    if(agent_->myaddr_ != hdrz->src_) {    // Only Non-source case...
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_IERP_REQUEST | -S %d -D %d | -IS %d -ID %d | -QID %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");}
                    } // [Log: End]

    // [CommonTask-1]: Check if this is a new query?? If not, Add this Query to the Cache
    Time now = Scheduler::instance().clock(); // get the time
    DetectedQuery *handleToFoundDQ = NULL;
    int foundQueryFlag;
    foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
    if(foundQueryFlag == TRUE) {
        if(hdrz->dest_ != agent_->myaddr_ && handleToFoundDQ->querySentFlag_ == TRUE) {    // Already sent the Query
            (agent_->pktUtil_).pkt_drop(p);    // Drop the original Packet 
            return;    // Nothing to Do
        }
    } else {    // For control Flooding...
        dqLst_.addQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, now+queryLifeTime_, &((agent_->iarpAgt_).pnLst_));
    }

    //如果我不是发送源
    // [CommonTask-2]: If I am not the Source, Do the Following...
    //如果我不是查询的发起者
    if(hdrz->src_ != agent_->myaddr_) {    // If I am not the originator of the query
        // 根据之前的 bordercaster 信息标记所有覆盖的外围节点。
        // [Sub-CommonTask-1]: Mark All covered peripheral nodes based on previous bordercaster info.
        foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
        assert(foundQueryFlag == TRUE);//一定会在该缓存中
        markCoveredPN(handleToFoundDQ, hdrz->lastbc_);
        //检查我是否是多播接收者
        //hdrz->mcLst_的添加方法在此处不明，可能在下面添加
        // [Sub-CommonTask-2]: Check if I am a Multicast receiver or not
        int receiveFlag = (agent_->pktUtil_).pkt_AmIMultiCastReciver(p, agent_->myaddr_);
        if(receiveFlag == FALSE) {
            return;// Nothing to Do
        }
    }
    //如果我是目标节点
    // [Task-1]: If I am the Destination [Create a ROUTE-REPLY(New Packet) and Send it back]]
    if(hdrz->dest_ == agent_->myaddr_) {
        //检查我发了多少replies
        // [SubTask-0]: Check how many replies have been sent.
        DetectedQuery *handleToFoundDQ = NULL;
        int foundQueryFlag;
        foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
        assert(foundQueryFlag == TRUE);
        if(handleToFoundDQ->totalReplySent_ >= DEFAULT_MAX_IERP_REPLY) {
            return; // No more replies should be sent.
        } else {
            handleToFoundDQ->totalReplySent_++;    // Reply is sent in below code.
        }
        //创建IERP_REPLY包
        // [SubTask-1]: Create a IERP_REPLY packet ['hdrz->routelength_+1': My address needs to be added]
        Packet *pnew = (agent_->pktUtil_).pkt_create(IERP_REPLY, hdrz->dest_, hdrz->routelength_+1);
        addMyAddressToRoute(p, pnew);    // Add My address to the route
        // [SubTask-2]: Set Parameters...
        hdr_zrp *hdrznew = HDR_ZRP(pnew);
        hdrznew->src_ = hdrz->src_;            // Identification fields at Sender of the Query
        hdrznew->dest_ = hdrz->dest_;            // Identification fields at Sender of the Query
        hdrznew->seq_ = hdrz->seq_;            // Identification fields at Sender of the Query
        hdrznew->queryID_ = hdrz->queryID_;        // Identification fields at Sender of the Query
        hdrznew->routeindex_ = hdrznew->routelength_-1;    // My_address in the route
        // [SubTask-3]: Send the Packet
        hdr_ip *hdripnew = HDR_IP(pnew);
        hdripnew->saddr() = agent_->myaddr_;
            hdripnew->daddr() = hdrznew->route_[hdrznew->routeindex_-1];    // Previous Hop
        (agent_->pktUtil_).pkt_send(pnew, hdripnew->daddr(), 0.00);
                    if(DEBUG) { // [Log: XMIT_IERP_REPLY]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REPLY | -S %d -D %d | -IS %d -ID %d | -QID %d | ",
                    agent_->myaddr_, now, hdrznew->src_, hdrznew->dest_, hdripnew->saddr(),
                    hdripnew->daddr(), hdrznew->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(pnew); agent_->print_tables(); printf("\n");
                    } // [Log: End]
        // 为查询发送设置标志。
        // [SubTask-4]: Set the flag for Query-Sent...
        handleToFoundDQ = NULL;
        foundQueryFlag = -1;
        foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
        assert(foundQueryFlag == TRUE);                // Query Must be in the cache
        assert(handleToFoundDQ->querySentFlag_ == FALSE);    // Query must not be sent before
        handleToFoundDQ->querySentFlag_ = TRUE;            // Set the flag for query-sent [Here, Query-processed]
        // [SubTask-5]: Drop the Packet...//丢弃包
        (agent_->pktUtil_).pkt_drop(p);    // Drop the original Packet 
        return;
    }

    // [任务 2]：如果 Destination 位于 My Zone 中.. [向 Destination 单播请求]
    // [Task-2]: If Destination lies in My Zone.. [Unicast the request to Destination]
    // This Node should multicast - too - to only 1 node [weird use of words ;)]
    InnerRoute *handleToFoundRoute = NULL;
    int foundRouteFlag, alreadySentNb=-1;
    //在自己的内部路由表查询该目标节点//返回查询标志
    foundRouteFlag = (agent_->iarpAgt_).irLst_.findRoute(hdrz->dest_, &handleToFoundRoute);
    if(foundRouteFlag == TRUE) {// Found Route in Zone//该节点在范围内
        // [SubTask-1]: Create a packet//创建包
        hdr_ip *hdrip = HDR_IP(p);
        int ttl = hdrip->ttl() - 1;    // Reducing the TTL value
        //创建IERP_REQUEST的包
        Packet *pnew = (agent_->pktUtil_).pkt_create(IERP_REQUEST, handleToFoundRoute->nextHop_, ttl);
        addMyAddressToRoute(p, pnew);    // Add My address to the route
        // [SubTask-2]: 设置参数
        hdr_zrp *hdrznew = HDR_ZRP(pnew);
        hdrznew->src_ = hdrz->src_;    // Identification fields at Sender of the Query
        hdrznew->dest_ = hdrz->dest_;    // Identification fields at Sender of the Query
        hdrznew->seq_ = hdrz->seq_;    // Identification fields at Sender of the Query
        hdrznew->queryID_ = hdrz->queryID_;// Identification fields at Sender of the Query
        hdrznew->lastbc_ = agent_->myaddr_;
        hdrznew->routeindex_ = hdrznew->routelength_ - 1;    // My_address in the route [Just Added b4]
        // [SubTask-3]: 发送
        hdr_ip *hdripnew = HDR_IP(pnew);
        hdripnew->saddr() = agent_->myaddr_;    
        hdripnew->daddr() = handleToFoundRoute->nextHop_;
        //该部分创建Muti-Cast链表
        (agent_->pktUtil_).pkt_add_ADDRESS_space(pnew,1);// Add Multi-cast Addresses [Allocate Memory]
        hdrznew->mcLstSize_ = 1;// 设置参数
        hdrznew->mcLst_[0] = handleToFoundRoute->nextHop_;// Adding multi-cast address
        //====================
        (agent_->pktUtil_).pkt_send(pnew, hdripnew->daddr(), 0.00);
        alreadySentNb = handleToFoundRoute->nextHop_;
                    if(DEBUG) { // [Log: XMIT_IERP_REQUEST]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REQUEST | -S %d -D %d | -IS %d -ID %d | -QID %d |",
                    agent_->myaddr_, now, hdrznew->src_, hdrznew->dest_, 
                    hdripnew->saddr(), hdripnew->daddr(), hdrznew->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(pnew); agent_->print_tables(); printf("\n");
                    } // [Log: End]
        // [SubTask-4]: Set the flag for Query-Sent...
        DetectedQuery *handleToFoundDQ = NULL;
        int foundQueryFlag;
        foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
        assert(foundQueryFlag == TRUE);                // Query Must be in the cache
        assert(handleToFoundDQ->querySentFlag_ == FALSE);    // Query must not be sent before
        handleToFoundDQ->querySentFlag_ = TRUE;            // Set the flag for query-sent
    }
    //广播Query
    // [Task-3]: Bordercast The Query... 
    // [SubTask-1]: Bordercast it(Find all Next-Hop neighbors to  un-covered-perpheral-nodes)...
    NeighborList NbLst; // Temporary list of query-forwarders
    PeripheralNode *curPN;
    handleToFoundRoute = NULL;
    foundRouteFlag = -1;
    handleToFoundDQ = NULL;
    foundQueryFlag = -1;    
    foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
    assert(foundQueryFlag == TRUE); // Query Must be in the cache
    curPN = (handleToFoundDQ->pnLst_).head_;
    for(int i=0; i<(handleToFoundDQ->pnLst_).numPerNodes_; i++) {    // For all Peripheral Nodes
        if(curPN->coveredFlag_ == TRUE) {
            curPN = curPN->next_;    // Advance Pointer
            continue;        // Skip this Peripheral Node
        }
        foundRouteFlag = (agent_->iarpAgt_).irLst_.findRoute(curPN->addr_, &handleToFoundRoute);
        if(foundRouteFlag == TRUE) {    // Add these Neighbors into List
            int foundNbFlag;
            Neighbor *handleToFoundNb = NULL;    // Do Not Replicate Same Neighbors
            foundNbFlag = NbLst.findNeighbor(handleToFoundRoute->nextHop_, &handleToFoundNb);
            if(foundNbFlag == FALSE && handleToFoundRoute->nextHop_ != hdrz->lastbc_
                        && handleToFoundRoute->nextHop_ != alreadySentNb) {
                // Last-2 args-doesnt matter
                NbLst.addNeighbor(handleToFoundRoute->nextHop_, 0.0, LINKDOWN); 
            }
        }
        curPN->coveredFlag_ = TRUE;    // We are sending the query    
        curPN = curPN->next_;
    }
    // [SubTask-2]: Bordercast it(Send a Query to every neighbor in the list)...
    if(NbLst.isEmpty() == FALSE) {
        hdr_ip *hdrip = HDR_IP(p);
        int ttl = hdrip->ttl() - 1;    // Reducing the TTL value
        // 1. Create a Packet... 
        Packet *pnew = (agent_->pktUtil_).pkt_create(IERP_REQUEST, IP_BROADCAST, ttl);
        // 2. Add My address to the route...
        addMyAddressToRoute(p, pnew);
        // 3. Add Multicast addresses...
        hdr_zrp *hdrznew = HDR_ZRP(pnew);
        (agent_->pktUtil_).pkt_add_ADDRESS_space(pnew, NbLst.numNeighbors_);// Add Multi-cast Addresses[Allocate Memory]
        hdrznew->mcLstSize_ = NbLst.numNeighbors_;    // Setting parameter
        Neighbor *curNb = NULL; 
        curNb = NbLst.head_;
        for(int i=0; i<NbLst.numNeighbors_; i++) {
            hdrznew->mcLst_[i] = curNb->addr_;
            curNb = curNb->next_;
        }
        // 4. Set parameters...
        hdrznew->src_ = hdrz->src_;    // Identification fields at Sender of the Query
        hdrznew->dest_ = hdrz->dest_;    // Identification fields at Sender of the Query
        hdrznew->seq_ = hdrz->seq_;    // Identification fields at Sender of the Query
        hdrznew->queryID_ = hdrz->queryID_;// Identification fields at Sender of the Query
        hdrznew->lastbc_ = agent_->myaddr_;
        hdrznew->routeindex_ = hdrznew->routelength_-1;    // My_address in the route [Just Added b4]

        // 5. Multicast(Broadcast) the Packet...
        hdr_ip *hdripnew = HDR_IP(pnew);
        hdripnew->saddr() = agent_->myaddr_;    
            hdripnew->daddr() = IP_BROADCAST;// Just to be sure
        (agent_->pktUtil_).pkt_broadcast(pnew, (double)(IERP_XMIT_JITTER));
                    if(DEBUG) { // [Log: XMIT_IERP_REQUEST]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REQUEST | -S %d -D %d | -IS %d -ID BROADCAST | "
                    "-QID %d | Sent to ", agent_->myaddr_, now, hdrznew->src_, hdrznew->dest_, 
                    hdripnew->saddr(), hdrznew->queryID_);
                    NbLst.printNeighbors(); (agent_->pktUtil_).pkt_print_route(pnew); 
                    agent_->print_tables(); printf("\n");
                    } // [Log: End]
        // 6. Set the flag for Query-Sent...
        DetectedQuery *handleToFoundDQ = NULL;
        int foundQueryFlag;
        foundQueryFlag = dqLst_.findQuery(hdrz->src_, hdrz->dest_, hdrz->queryID_, &handleToFoundDQ);
        assert(foundQueryFlag == TRUE);                // Query Must be in the cache
        assert(handleToFoundDQ->querySentFlag_ == FALSE);    // Query must not be sent before
        handleToFoundDQ->querySentFlag_ = TRUE;            // Set the flag for query-sent
    }
    (agent_->pktUtil_).pkt_drop(p);    // Drop the original Packet 
}
```

---

#### 4.1.9 IERPAgent类内方法addLinkStateFromRoute()

当从路由回复消息中提取链路状态信息时调用此函数。这链路状态信息从源路由中提取并添加到链路状态表中。

```cpp
//该函数再次更新link-status情报
int 
IERPAgent::addLinkStateFromRoute(nsaddr_t *route, int size) {
    // For each link in route, add them in the IARP-Link-State-Table
    //对于路由中的每个链接，将它们添加到 IARP-Link-State-Table
    assert(size!=0);
    Time now = Scheduler::instance().clock(); // get the time
    LinkState *handleToFoundLS;
    int LSFoundFlag, LSTChangeFlag=FALSE;
    for(int i=0; i<size-1; i++) {
        handleToFoundLS = NULL; LSFoundFlag = FALSE;
        LSFoundFlag = (agent_->iarpAgt_).lsLst_.findLink(route[i], route[i+1], &handleToFoundLS);
        if(LSFoundFlag == FALSE) {    // New one
            (agent_->iarpAgt_).lsLst_.addLink(route[i], route[i+1], -1, LINKUP, now+routeLifeTime_);
            LSTChangeFlag = TRUE;
        } else {            // Existing One
            if(handleToFoundLS->isup_ == LINKDOWN) {
                LSTChangeFlag = TRUE;
            }
            handleToFoundLS->isup_ = LINKUP;        // Update Attributes - Link-State
            handleToFoundLS->expiry_ = now+routeLifeTime_;// Update Attributes - Expiry Time
        }
    }

    // If there is a change in Link-State-Table, then rebuild the routing table.
    if(LSTChangeFlag == TRUE) {
        (agent_->iarpAgt_).buildRoutingTable();          // Rebuild Routing Table
        //agent_->route_SendBuffer_pkt();    // Send Buffered packets... [Its done at callee function]
    }

    return LSTChangeFlag;
}
```

---

#### 4.1.10 IERPAgent类内方法removeLinkStateFromBrokenRoute()

当从路由错误消息中提取损坏的链路状态信息时调用此函数。断开的链路状态信息从源路由中提取并添加到链路状态表中。

```cpp
int 
IERPAgent::removeLinkStateFromBrokenRoute(nsaddr_t lnkSrc, nsaddr_t lnkDest) {
    // Remove the broken link from the link-state-list if it exists.
    LinkState *handleToFoundLS=NULL;
    int LSFoundFlag=FALSE;
    LSFoundFlag = (agent_->iarpAgt_).lsLst_.findLink(lnkSrc, lnkDest, &handleToFoundLS);

    // If there is a change in Link-State-Table, then rebuild the routing table.
    if(LSFoundFlag  == TRUE) {
        handleToFoundLS->isup_ = LINKDOWN;      // Set the link-stauts = DOWN 
        (agent_->iarpAgt_).buildRoutingTable();          // Rebuild Routing Table
    }

    return LSFoundFlag;
}
```

---

#### 4.1.11IERPAgent类内方法recv_IERP_ROUTE_REPLY()

```cpp
/* 6. Receiving IERP-route-reply. */
//接受IERP-route-reply
void 
IERPAgent::recv_IERP_ROUTE_REPLY(Packet* p) {
    hdr_ip *hdrip = HDR_IP(p);
    hdr_cmn *hdrc = HDR_CMN(p);
    hdr_zrp *hdrz = HDR_ZRP(p);
                    if(DEBUG) { // [Log: RECV_IERP_REPLY]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_IERP_REPLY | -S %d -D %d | -IS %d -ID %d | -QID %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->queryID_);    
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    //获取当前时间
    Time now = Scheduler::instance().clock();
    // [Task-1]: (IF) I am the Source.. [Store the route in OUTER-ROUTE-LIST - 
    //    2-Cases: 1) Enter New Route; OR 2)Replace the old One(If new one is better)]
    // 放入新的路由或者替换掉旧的(如果你有新的更好的)
    // 意味着我是发送源节点，我已经发送过该Query
    if(hdrz->src_ == agent_->myaddr_) {    // MEANS - I have intitated the Query
        //更新拓扑表
        // [SubTask-1]: Adding this info to topology table...
        int routingTableUpdatedFlag=FALSE;
        routingTableUpdatedFlag = addLinkStateFromRoute(hdrz->route_, hdrz->routelength_);
        if(routingTableUpdatedFlag == TRUE) {
            agent_->route_SendBuffer_pkt();    // Send Buffered packets...
        }
        //从已经送信QueryBuffer中删除该Query，因为已经完成
        // [SubTask-2]: Delete the query from Sent_Query_List
        SentQuery *handleToFoundSQ = NULL;
        int foundSQFlag;
        foundSQFlag = sqLst_.findQuery(hdrz->src_, hdrz->dest_, &handleToFoundSQ);
        if(foundSQFlag == TRUE) {    // Remove the Entry
            sqLst_.removeQuery(hdrz->src_, hdrz->dest_);
        }
        // 丢掉该包
        // [SubTask-3]: Drop the packet
        (agent_->pktUtil_).pkt_drop(p);
    // [Task-2]: (ELSE) Forward it along the Route (Backwards) [No need to generate new packet]
    // 继续转发该包，不需要产生新的
    } else {
        // 转发回复.. 将此信息添加到拓扑表中。
        // Snooping the forwarding replies.. Adding this info into topology table.
        if(IERP_REPLY_SNOOP) {
            int routingTableUpdatedFlag=FALSE;
            routingTableUpdatedFlag = addLinkStateFromRoute(hdrz->route_, hdrz->routelength_);
            if(routingTableUpdatedFlag == TRUE) {
                agent_->route_SendBuffer_pkt();    // Send Buffered packets...
            }
        }
        //沿路由转发（向后）
        // Forward it along the Route (Backwards) [No need to generate new packet]
        hdrz->routeindex_--;    // Move 1 hop (PREV) into the route(Should be My address)
        assert(hdrz->route_[hdrz->routeindex_] == agent_->myaddr_); assert(hdrz->routeindex_-1 >= 0);
        hdrc->direction() = hdr_cmn::DOWN;
        hdrip->saddr() = agent_->myaddr_;
        hdrip->daddr() = hdrz->route_[hdrz->routeindex_-1];    // Previous hop in the route    
        (agent_->pktUtil_).pkt_send(p, hdrip->daddr(), 0.00);
                    if(DEBUG) { // [Log: XMIT_IERP_REPLY]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | XMIT_IERP_REPLY | -S %d -D %d | -IS %d -ID %d | -QID %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->queryID_);
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    }
}
```

---

#### 4.1.12 IERPAgent类内方法recv_IERP_ROUTE_ERROR()

```cpp
/* 7. Receiving IERP-route-error. */
//接收到IERP-route-error包
void
IERPAgent::recv_IERP_ROUTE_ERROR(Packet* p) {
    // [Task-1]: (IF) I am the Source.. [Discard the route in OUTER-ROUTE-LIST]
    hdr_cmn *hdrc = HDR_CMN(p);
    hdr_zrp *hdrz = HDR_ZRP(p);
    hdr_ip *hdrip = HDR_IP(p);
                    if(DEBUG) { // [Log: RECV_IERP_ERROR]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_IERP_ERROR | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->seq_);
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    if(hdrz->src_ == agent_->myaddr_) {    // I am the Source...
        // [SubTask-1]: Remove the broken link info from topology table.
        removeLinkStateFromBrokenRoute(hdrz->links_[0].src_, hdrz->links_[0].dest_);
        // [SubTask-2]: Drop the packet
        (agent_->pktUtil_).pkt_drop(p);
    // [Task-2]: (ELSE) Forward it along the Route (Backwards) [No need to generate new packet]
    } else {
        if(IERP_ERROR_SNOOP) {
            // Remove the broken link info from topology table.
            removeLinkStateFromBrokenRoute(hdrz->links_[0].src_, hdrz->links_[0].dest_);
        }
        hdrz->routeindex_--;    // Move 1 hop (PREV) into the route(Should be My address)
        assert(hdrz->route_[hdrz->routeindex_] == agent_->myaddr_); assert(hdrz->routeindex_-1 >= 0);
        hdrc->direction() = hdr_cmn::DOWN;
        hdrip->saddr() = agent_->myaddr_;
        hdrip->daddr() = hdrz->route_[hdrz->routeindex_-1];
        (agent_->pktUtil_).pkt_send(p, hdrip->daddr(), 0.00);
                    if(DEBUG) { // [Log: RECV_IERP_ERROR]
                    Time now = Scheduler::instance().clock(); // get the time
                    printf("\n_%d_ [%6.6f] | RECV_IERP_ERROR | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
                    agent_->myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(),
                    hdrz->seq_);
                    (agent_->pktUtil_).pkt_print_route(p); agent_->print_tables(); printf("\n");
                    } // [Log: End]
    }
}
```

---

## 5.ZRP(ZONE ROUTING PROTOCOL)

### 5.1 ZRP的包头类

```cpp
/* [SECTION-5]------------------------------------------------------------------
 *    1) ZRP HEADER STRUCTURE
 * [SUB-SECTION-5.1]------------------------------------------------------------
 * ZRP HEADER STRUCTURE:-
 * -----------------------------------------------------------------------------
 */
class LSU;    // Pre-Declaration
//zrp的包头
struct hdr_zrp {
    // Common Attributes to All packets... [Default Size = 10 bytes]
    int zrptype_;         // Packet type                    [1 byte]
    Time pktsent_;        // Packet Sending time                [Logically not sent]
    int radius_;        // Sender's Zone Radius (Maybe Useful in Future)[Not Used]
    int seq_;             // Sequence number of a packet            [1 byte]
    nsaddr_t src_;        // 32-bit Address                [4 bytes]
    nsaddr_t dest_;        // 32-bit Address                [4 bytes]

    // IARP Specific...            [IARP size = 2 + 5*links]
    int forwarded_;      // TRUE if forwarded before            [1 byte]
    LSU *links_;          // pointer to link state list             [5 bytes per link - address(4) + link(1)]
    int numlinks_;        // Number of links in packet            [1 byte]

    // IERP Specific...            [IERP size (for REQUEST) = 7 + 4*routelength]
    //                    [    (for REPLY) = 3 + 4*routelength]
    //                    [    (for ERROR) = 2 + 5(for link info) + 4*routelength] 
    //                    [    (for DATA) = 2 + 4*routelength]
    nsaddr_t *mcLst_;    // List of addresses to relay(Multicast) the    [Not Used]
                        // route-request query
    int mcLstSize_;        // Size of above list                [Not Used]
    nsaddr_t lastbc_;    // 32-bit Address [Not Used much]        [REQUEST - 4 bytes]
    nsaddr_t *route_;     // pointer to route list data            [REQUEST/REPLY/ERROR/DATA 4 bytes per address]
    int routelength_;    // Route Length of IERP route stored in route_    [REQUEST/REPLY/ERROR/DATA 1 byte]
      int routeindex_;      // Pointer to a current route node in route_    [REQUEST/REPLY/ERROR/DATA 1 byte]
    int queryID_;          // IERP query id counter            [REQUEST/REPLY 1 byte]

      // this is where the original data for upper layer pkts
      // is saved while ZRP routes pkt, at dest this is placed
      // back into hdrip->dport(), ie this is part of encapsulated data
      int enc_dport_;
      int enc_daddr_;
      packet_t enc_ptype_;

    // Calculate the size of the header
    inline int size() {
        int s=0;
        switch(zrptype_) {
            case NDP_BEACON:
                s = ZRP_DEFAULT_HDR_LEN;
            break;
            case NDP_BEACON_ACK:
                s = ZRP_DEFAULT_HDR_LEN;
            break;
            case IARP_UPDATE:
                s = ZRP_DEFAULT_HDR_LEN + 2 + 5*numlinks_;
            break;
            case IERP_REPLY:
                s = ZRP_DEFAULT_HDR_LEN + 3 + 4*routelength_;
            break;
            case IERP_REQUEST:
                s = ZRP_DEFAULT_HDR_LEN + 7 + 4*routelength_;
            break;
            case IERP_ROUTE_ERROR:
                s = ZRP_DEFAULT_HDR_LEN + 7 + 4*routelength_;
            break;
            case IARP_DATA:
            case IERP_DATA:
                s = ZRP_DEFAULT_HDR_LEN + 2 + 4*routelength_;
            break;
        }
        return s;
    }

      // Packet header access functions
      static int offset_;
      inline static int& offset() { return offset_; }
      inline static hdr_zrp* access(const Packet* p) {
            return (hdr_zrp*) p->access(offset_);
      }
};
//链路状态
class LSU
{
  public:    
    nsaddr_t src_;
    nsaddr_t dest_;
    int isUp_;

    // Constructors...
    LSU() : src_(-1), dest_(-1), isUp_(LINKDOWN) {}    // Invalid Entries...
    LSU(nsaddr_t src, nsaddr_t dest, int isUp) : src_(src), dest_(dest),
        isUp_(isUp) {}
};
```

---

##### 5.1.1 ZRP的包头类的内部方法

```cpp
/* A supporting function used in finding the ZRP header in a packet. */
int hdr_zrp::offset_;
static class ZRPHeaderClass : public PacketHeaderClass {
  public:
  	ZRPHeaderClass() : PacketHeaderClass("PacketHeader/ZRP", sizeof(hdr_zrp)) {
          	bind_offset(&hdr_zrp::offset_);
	}
	void export_offsets() {
		field_offset("zrptype_", OFFSET(hdr_zrp, zrptype_));
		field_offset("src_", OFFSET(hdr_zrp, src_));
		field_offset("dest_", OFFSET(hdr_zrp, dest_));
		field_offset("seq_", OFFSET(hdr_zrp, seq_));
		field_offset("queryID_", OFFSET(hdr_zrp, queryID_));
		field_offset("lastbc_", OFFSET(hdr_zrp, lastbc_));
	}
}class_zrp_hdr;

/* A binding between ZRP and TCL, you will most likely not change this. */
static class ZRPClass : public TclClass {
  public:
	ZRPClass() : TclClass("Agent/ZRP") {}
	TclObject* create(int argc, const char*const* argv) {
		return(new ZRPAgent((nsaddr_t) atoi(argv[4])));
		// Tcl code will attach me and then pass in addr to me
		// see tcl/lib/ns-lib.tcl, under create-zrp-agent,
		// is done by "set ragent [new Agent/ZRP [$node id]]"
	}
} class_zrp;
```

### 5.2 ZRP的PacketUtil类

```cpp
 /* [SUB-SECTION-5.2]------------------------------------------------------------
 * PACKET UTILITIES CLASS:- PACKET RELATED UTILITIES
 * -----------------------------------------------------------------------------
 */
class PacketUtil
{
  public:
    // Data...
    int seq_;
    ZRPAgent *agent_;
    int startup_jitter_;

    PacketUtil(ZRPAgent *agent) : seq_(0), agent_(agent),
        startup_jitter_(DEFAULT_STARTUP_JITTER) {}

    // Methods...
    Packet* pkt_create(ZRPTYPE zrp_type, nsaddr_t addressee, int ttl); 
      void pkt_copy(Packet *pfrom, Packet *pto);
    void inc_seq() { seq_++; if (seq_ > MAX_SEQUENCE_ID) seq_ = 1;}
    void pkt_send(Packet *p, nsaddr_t addressee, Time delay);
    void pkt_broadcast(Packet *p, Time randomJitter);
    void pkt_add_LSU_space(Packet *p, int size);
    void pkt_free_LSU_space(Packet *p);
    void pkt_add_ROUTE_space(Packet *p, int size);
    void pkt_free_ROUTE_space(Packet *p);
    void pkt_add_ADDRESS_space(Packet *p, int size);
    void pkt_free_ADDRESS_space(Packet *p);
    int pkt_AmIMultiCastReciver(Packet *p, nsaddr_t addressToCheck);
    int pkt_AmIOnTheRoute(Packet *p, nsaddr_t addressToCheck);
    void pkt_print_links(Packet *p);
    void pkt_print_route(Packet *p);
    void pkt_drop(Packet *p);
};
```

---

#### 5.2.1  PacketUtil类的内部方法pkt_create()

```cpp
/* 1. Creates a fresh new packet and initializes as much as it knows how. */
Packet* 
PacketUtil::pkt_create(ZRPTYPE zrp_type, nsaddr_t addressee, int ttl)
{
	Time now = Scheduler::instance().clock(); // get the time
  	Packet *p = agent_->Myallocpkt(); // fresh new packet

	hdr_ip *hdrip = HDR_IP(p);
  	hdr_cmn *hdrc = HDR_CMN(p);
  	hdr_zrp *hdrz = HDR_ZRP(p);

	// Common Header
  	hdrc->ptype() = PT_ZRP;   	// ZRP pkt type
  	hdrc->next_hop() = addressee;  
	hdrc->direction() = hdr_cmn::DOWN;	// Sending packets DOWN
  	hdrc->addr_type_ = NS_AF_NONE;
  	hdrc->size() = IP_HDR_LEN;  	//  set default packet size
	// IP Header
  	hdrip->ttl() = ttl;
  	hdrip->saddr() = agent_->myaddr_;       // source address
  	hdrip->sport() = ROUTER_PORT;   // source port
  	hdrip->daddr() = addressee;     // dest address
  	hdrip->dport() = ROUTER_PORT;   // dest port
	// ZRP Header
  	hdrz->zrptype_ = zrp_type;      // which zrp pkt am I?
	hdrz->pktsent_ = now;		// Packet Sending Time
	hdrz->radius_ = agent_->radius_;// Sender's Radius
  	hdrz->seq_ = seq_;              // copy from gobal sequence counter
  	hdrz->forwarded_ = 0;           // have not been forwarded before
  	hdrz->src_ = agent_->myaddr_;   // I am originator, this is used by NDP/IARP
	hdrz->dest_ = addressee;	// dest address
	hdrz->links_ = NULL;		// Nothing in IARP Update
	hdrz->numlinks_ = 0;		//	"	"	 		
	hdrz->mcLst_ = NULL;		// Addresses to Multicast
	hdrz->mcLstSize_ = 0;		// Number of addresses to Multicast
	hdrz->lastbc_ = -1;		// '-1' is an invalid address 
	hdrz->route_ = NULL;		// Nothing in IERP Route
	hdrz->routelength_ = 0;		// 	" 	"
  	hdrz->routeindex_ = 0;          // where in the route list am I sending ?
	hdrz->queryID_ = -1;		// Invalid Query ID

	inc_seq(); 			// increments global sequence counter

	return(p);
}
```

----

#### 5.2.2 PacketUtil类的内部方法pkt_copy()

```cpp
/* 2. Copy an entire packet to another. */
void
PacketUtil::pkt_copy(Packet *pfrom, Packet *pto) {
  	hdr_cmn *hdrcfrom = HDR_CMN(pfrom);
  	hdr_ip *hdripfrom = HDR_IP(pfrom);
  	hdr_zrp *hdrzfrom = HDR_ZRP(pfrom);

  	hdr_cmn *hdrcto = HDR_CMN(pto);
  	hdr_ip *hdripto = HDR_IP(pto);
  	hdr_zrp *hdrzto = HDR_ZRP(pto);

	// Common Header
  	hdrcto->direction() = hdrcfrom->direction();
  	hdrcto->ptype() = hdrcfrom->ptype();
  	hdrcto->next_hop() = hdrcfrom->next_hop();
  	hdrcto->addr_type_ = hdrcfrom->addr_type_;
  	hdrcto->size() = hdrcfrom->size() ;
	// IP Header
  	hdripto->ttl() = hdripfrom->ttl();
  	hdripto->saddr() = hdripfrom->saddr();
  	hdripto->sport() = hdripfrom->sport();
  	hdripto->daddr() =  hdripfrom->daddr();
  	hdripto->dport() = hdripfrom->dport();
	// ZRP Header
  	hdrzto->zrptype_ = hdrzfrom->zrptype_;
  	hdrzto->pktsent_ = hdrzfrom->pktsent_;
  	hdrzto->radius_ = hdrzfrom->radius_;
  	hdrzto->seq_ = hdrzfrom->seq_;
  	hdrzto->forwarded_ = hdrzfrom->forwarded_;
  	hdrzto->src_ = hdrzfrom->src_;
  	hdrzto->dest_ = hdrzfrom->dest_;
  	hdrzto->numlinks_ = hdrzfrom->numlinks_;
	hdrzto->mcLstSize_ = hdrzfrom->mcLstSize_;
  	hdrzto->routelength_ = hdrzfrom->routelength_;
  	hdrzto->lastbc_ = hdrzfrom->lastbc_;
	hdrzto->queryID_ = hdrzfrom->queryID_;
	hdrzto->routeindex_ = hdrzfrom->routeindex_;
  	hdrzto->enc_dport_ = hdrzfrom->enc_dport_;
  	hdrzto->enc_daddr_ = hdrzfrom->enc_daddr_;
  	hdrzto->enc_ptype_ = hdrzfrom->enc_ptype_;
  	
	// Filling LSUs...
	pkt_add_LSU_space(pto, hdrzfrom->numlinks_);		// If zero, No memory is allocated
	for(int i=0; i<hdrzfrom->numlinks_; i++) {	
		hdrzto->links_[i].src_ = hdrzfrom->links_[i].src_;
		hdrzto->links_[i].dest_ = hdrzfrom->links_[i].dest_;
		hdrzto->links_[i].isUp_ = hdrzfrom->links_[i].isUp_;	
	}
	
	// Filling Route...
	pkt_add_ROUTE_space(pto, hdrzfrom->routelength_); 	// If zero, No memory is allocated
	for(int i=0; i<hdrzfrom->routelength_; i++) {	
		hdrzto->route_[i] = hdrzfrom->route_[i];
	}
	
	// Filling Multi-Cast List
	pkt_add_ADDRESS_space(pto, hdrzfrom->mcLstSize_);	// If zero, No memory is allocated
	for(int i=0; i<hdrzfrom->mcLstSize_; i++) {
		hdrzto->mcLst_[i] = hdrzfrom->mcLst_[i];
	}
}
```

---

#### 5.2.3 PacketUtil类的内部方法pkt_send()

```cpp
/* 3. Send packet, ie schedule it for sometime in future to target_ of ZRP, 
   which is handled by LL. This is unicast only to "addressee" */
void 
PacketUtil::pkt_send(Packet* p, nsaddr_t addressee, Time delay) {
  	hdr_ip *hdrip = HDR_IP(p);
  	hdr_cmn *hdrc = HDR_CMN(p);
	hdr_zrp *hdrz = HDR_ZRP(p);
	// 1. For Mac Failure
	hdrc->xmit_failure_ = zrp_xmit_failure_callback;
	hdrc->xmit_failure_data_ = (void*)agent_;	
  	// 2. set next hop and dest to addressee
	if(hdrz->zrptype_ == IARP_DATA || hdrz->zrptype_ == IERP_DATA) {
		if(agent_->myaddr_ == hdrz->src_) {
			hdrc->size() += hdrz->size();
		}
	}
	else {
		hdrc->size() = IP_HDR_LEN + hdrz->size();	// Adding the header length...
	}
  	hdrc->next_hop() = addressee;	// UNICAST...
  	hdrip->daddr() = addressee;	// UNICAST...
  	// 3. transmit, no delay
	agent_->XmitPacket(p, delay);	// No jitter
	agent_->tx_++; 			// increment pkt transmitted counter
}
```

---

#### 5.2.4 PacketUtil类的内部方法pkt_broadcast()

```cpp
/* 4.Broadcast packet to all neighbors. */
void 
PacketUtil::pkt_broadcast(Packet *p, Time randomJitter) {
	hdr_ip *hdrip = HDR_IP(p);
	hdr_cmn *hdrc = HDR_CMN(p);
	hdr_zrp *hdrz = HDR_ZRP(p);
  	// 1. set next hop and dest to addressee
	if(hdrz->zrptype_ == IARP_DATA || hdrz->zrptype_ == IERP_DATA) {
		if(agent_->myaddr_ == hdrz->src_) {
			hdrc->size() += hdrz->size();
		}
	}
	else {
		hdrc->size() = IP_HDR_LEN + hdrz->size();	// Adding the header length...
	}
	hdrc->next_hop() = IP_BROADCAST;// BROADCAST...
	hdrip->daddr() = IP_BROADCAST;	// BROADCAST...
  	// 2. transmit, with delay
	agent_->XmitPacket(p, randomJitter);
  	agent_->tx_++; 			// increment pkt transmitted counter
}
```

---

#### 5.2.5 PacketUtil类的内部方法pkt_add_LSU_space()

```cpp
/* 5. Adds Link-space in the Packet. */
void 
PacketUtil::pkt_add_LSU_space(Packet *p, int size) {
	// 1. If some previous allocation is there then delete it...
  	hdr_zrp *hdrz = HDR_ZRP(p);
	if (hdrz->links_ != NULL) {
		delete [] (hdrz->links_);
		hdrz->links_ = NULL;
	}
			
	// 2. Allocate Memory...
	if(size>0) {
		hdrz->links_ = new LSU[size];
		if(hdrz->links_==NULL) {
			printf("### Memory Allocation Error in [PacketUtil::pkt_add_LSU_space] ###");
		}
	}
}
```

---

#### 5.2.6 PacketUtil类的内部方法pkt_free_LSU_space()

```cpp
/* 6. Frees Link-space in the Packet. [Not using because of memory corruption at MAC layer] */
void 
PacketUtil::pkt_free_LSU_space(Packet *p) {
	/*// If some previous allocation is there then delete it...
  	hdr_zrp *hdrz = HDR_ZRP(p);
	if (hdrz->links_ != NULL) {
		delete [] (hdrz->links_);
		hdrz->links_ = NULL;
	}*/
}
```

---

#### 5.2.7 PacketUtil类的内部方法pkt_add_ROUTE_space()

```cpp
/* 7. Adds Route-space in the Packet. */
void 
PacketUtil::pkt_add_ROUTE_space(Packet *p, int size) {
	// 1. If some previous allocation is there then delete it...
  	hdr_zrp *hdrz = HDR_ZRP(p);	
	if (hdrz->route_ != NULL) {
		delete [] (hdrz->route_);
		hdrz->route_ = NULL;
	}
			
	// 2. Allocate Memory...
	if(size>0) {
		hdrz->route_ = new nsaddr_t[size];
		if(hdrz->route_==NULL) {
			printf("### Memory Allocation Error in [PacketUtil::pkt_add_ROUTE_space] ###");
		}
	}
}
```

---

#### 5.2.8 PacketUtil类的内部方法pkt_free_ROUTE_space()

```cpp
/* 8. Frees Route-space in the Packet. */
void 
PacketUtil::pkt_free_ROUTE_space(Packet *p) {
	/*// If some previous allocation is there then delete it...
  	hdr_zrp *hdrz = HDR_ZRP(p);	
	if (hdrz->route_ != NULL) {
		delete [] (hdrz->route_);
		hdrz->route_ = NULL;
	}*/
}
```

----

#### 5.2.9 PacketUtil类的内部方法pkt_add_ADDRESS_space()

```cpp
/* 9. Adds Multicast-Address-space in the Packet. */
void 
PacketUtil::pkt_add_ADDRESS_space(Packet *p, int size) {
	// 1. If some previous allocation is there then delete it...
  	hdr_zrp *hdrz = HDR_ZRP(p);	
	if (hdrz->mcLst_ != NULL) {
		delete [] (hdrz->mcLst_);
		hdrz->mcLst_ = NULL;
	}
			
	// 2. Allocate Memory...
	if(size>0) {
		hdrz->mcLst_ = new nsaddr_t[size];
		if(hdrz->mcLst_==NULL) {
			printf("### Memory Allocation Error in [PacketUtil::pkt_add_ADDRESS_space] ###");
		}
	}
}
```

---

#### 5.2.10 PacketUtil类的内部方法pkt_free_ADDRESS_space()

```cpp
/* 10. Frees Multicast-Address-space in the Packet. */
void 
PacketUtil::pkt_free_ADDRESS_space(Packet *p) {
	/*// If some previous allocation is there then delete it...
  	hdr_zrp *hdrz = HDR_ZRP(p);	
	if (hdrz->mcLst_ != NULL) {
		delete [] (hdrz->mcLst_);
		hdrz->mcLst_ = NULL;
	}*/
}
```

---

#### 5.2.11 PacketUtil类的内部方法pkt_AmIMultiCastReciver()

```cpp
/* 11. Checks if 'addressToCheck' is on Multicast list or not. */
int 
PacketUtil::pkt_AmIMultiCastReciver(Packet *p, nsaddr_t addressToCheck) {
	hdr_zrp *hdrz = HDR_ZRP(p);
	// 1. If list is EMPTY, return FALSE.
	if(hdrz->mcLstSize_ == 0) {
		return FALSE;
	}
	// 2. Search for the address.
	int foundFlag = FALSE;
	for(int i=0; i<hdrz->mcLstSize_; i++) {
		if(hdrz->mcLst_[i] == addressToCheck) {
			foundFlag = TRUE;
			break;
		}
	}
	return foundFlag;
}
```

---

#### 5.2.12 PacketUtil类的内部方法pkt_AmIOnTheRoute()

```cpp
/* 12. Find if the asked Address is in the route of the packet or not. */
int 
PacketUtil::pkt_AmIOnTheRoute(Packet *p, nsaddr_t addressToCheck) {
	hdr_zrp *hdrz = HDR_ZRP(p);
	int foundFlag=FALSE;
	if(hdrz->routelength_ == 0) {
		return FALSE;
	}

	for(int i=0; i<hdrz->routelength_; i++) {
		if(addressToCheck == hdrz->route_[i]) {
			foundFlag = TRUE;
			break;
		}
	}
	return foundFlag;
}
```

---

#### 5.2.13 PacketUtil类的内部方法pkt_print_links()

```cpp
/* 13. Print all links contained in the Packet. */
void 
PacketUtil::pkt_print_links(Packet *p) {
	hdr_zrp *hdrz = HDR_ZRP(p);
	printf("| Packet contains Links: ");
	if(hdrz->numlinks_ <= 0) {
		printf("None. |");
	} else {
		for(int i=0; i<hdrz->numlinks_; i++) {
			if (hdrz->links_[i].isUp_ == LINKUP) {	// UP link
				printf("[%d=%d], ",hdrz->links_[i].src_, hdrz->links_[i].dest_); 
			} else {				// DOWN link
				printf("[%d#%d], ",hdrz->links_[i].src_, hdrz->links_[i].dest_); 
			}
		}
		printf("|");
	}
}
```

---

#### 5.2.14 PacketUtil类的内部方法pkt_print_route()

```cpp
/* 14. Print route contained in the Packet.*/
void 
PacketUtil::pkt_print_route(Packet *p) {
	hdr_zrp *hdrz = HDR_ZRP(p);
	printf("| Packet contains Route: [");
	if(hdrz->routelength_ <= 0) {
		printf("Empty] |");
	} else {
		for(int i=0; i<hdrz->routelength_; i++) {
			if(hdrz->routeindex_ == i) {	// Highlighting Route_index field
				printf(" (%d)", hdrz->route_[i]);
			} else {
				printf(" %d", hdrz->route_[i]);
			}
		}
		printf("] |");
	}
}
```

---

#### 5.2.15 PacketUtil类的内部方法pkt_drop()

```cpp
/* 15. Drop the Packet. */
void 
PacketUtil::pkt_drop(Packet *p) {
	Packet::free(p);
	/*hdr_zrp *hdrz = HDR_ZRP(p);
	switch(hdrz->zrptype_) {
		case NDP_BEACON:
		case NDP_BEACON_ACK:
			Packet::free(p);
		break;	
	}*/
}
```

---

### 5.3 ZRP的sendBuffer类

该类存储还未发送的包

```cpp
/* [SUB-SECTION-5.3]------------------------------------------------------------
 * SEND BUFFER:- BUFFER TO STORE UN-DELIVERED UPPER-LAYER PACKETS
 * -----------------------------------------------------------------------------
 */
class SendBufferEntry
{
  public:
	Packet *pkt_;		// Packet to Send from Upper-Layer
	nsaddr_t dest_; 	// Address of Destination
	Time expiry_;		// Time to expire the packet
	SendBufferEntry *next_;	// Link to next entry in the list

	SendBufferEntry() : pkt_(NULL), dest_(-1), expiry_(0.00), next_(NULL) {}
	SendBufferEntry(Packet *pkt, nsaddr_t dest, Time expiry) : pkt_(pkt), 
		dest_(dest), expiry_(expiry), next_(NULL) {}
};

class SendBuffer
{
  public:
	ZRPAgent *agent_;
	SendBufferEntry *head_;
	int numPackets_;

	SendBuffer(ZRPAgent *agent) : agent_(agent), head_(NULL), numPackets_(0) {}

	// Methods...
	void addPacket(Packet *pkt, nsaddr_t dest, Time expiry);
	void purgeExpiredPackets();
	void freeList();
};
```

---

#### 5.3.1 sendBuffer类内部方法addPacket()

```cpp
/* 1. Add Packet at the head. */
void 
SendBuffer::addPacket(Packet *pkt, nsaddr_t dest, Time expiry) {
	SendBufferEntry *newPkt = new SendBufferEntry(pkt, dest, expiry); // Create a new Entry
	if(newPkt == NULL) {	// Check for Allocation Error
		printf("### Memory Allocation Error in [SendBuffer::addPacket] ###");
		exit(0);
	}
	newPkt->next_ = head_;	// Made necessary Joinings			
	head_ = newPkt;
	numPackets_++;		// Increment Number of Packets
}

```

---

#### 5.3.2 sendBuffer类内部方法purgeExpiredPackets()

```cpp
/* 2. Remove all Expired-Packets. */
void 
SendBuffer::purgeExpiredPackets() {
	if(numPackets_ == 0) {
		return;		// Nothing to do
	}
	// [case-1]: Leaving the 1st case(head_ case)
	SendBufferEntry *prev, *cur;
	prev = head_;
	cur = head_->next_;
  	Time now = Scheduler::instance().clock(); 	// get the time
	for(int i=1; cur!=NULL; i++) {
		if(cur->expiry_<now) { 		// Delete the Expired-Packet
			prev->next_ = cur->next_;
			SendBufferEntry *toBeDeleted = cur;
			cur = cur->next_;
			assert(toBeDeleted->pkt_ != NULL);
			agent_->Mydrop(toBeDeleted->pkt_, DROP_RTR_QTIMEOUT);	// drops the packet
			//Packet::free(toBeDeleted->pkt_); // Not needed anymore
			delete toBeDeleted;
			numPackets_--;		// Decrement Number of Packets
		} else {
			prev = cur;		// Advance the Pointers
			cur = cur->next_;				
		}
	}
	
	// [case-2]: head_ case...
	if(head_!=NULL) {
		if(head_->expiry_<now) {
			SendBufferEntry *toBeDeleted = head_;
			head_ = head_->next_;
			assert(toBeDeleted->pkt_ != NULL);
			agent_->Mydrop(toBeDeleted->pkt_, DROP_RTR_QTIMEOUT);	// drops the packet
			//Packet::free(toBeDeleted->pkt_); // Not needed anymore
			delete toBeDeleted;
			numPackets_--;		// Decrement Number of Packets
		}
	}
	
}
```

---

#### 5.3.3 sendBuffer类内部方法freeList()

```cpp
/* 3. Empty the Buffer. */
void 
SendBuffer::freeList() {
	if(numPackets_==0) {
		return;
	}
	
	SendBufferEntry *cur, *toBeDeleted;
	cur = head_;
	for(int i=0; i<numPackets_; i++) {
		toBeDeleted = cur;
		assert(toBeDeleted->pkt_ != NULL);
		Packet::free(toBeDeleted->pkt_); // Not needed anymore
		cur = cur->next_;
		delete toBeDeleted;
	}

	head_ = NULL;
	numPackets_ = 0;
}

```

---

### 5.4 ZRPAgent类

```cpp
/* [SUB-SECTION-5.4]------------------------------------------------------------
 * ZRP AGENT:- ZONE ROUTING AGENT
 * -----------------------------------------------------------------------------
 */
class ZRPAgent : public Agent {
 public:
	// Tcl Related...
	char*  myid_;   	// (char *)myid_ is string equiv of (int)myaddr_
	PriQueue* ll_queue;
	Trace* tracetarget;
	MobileNode* node_;
	NsObject* port_dmux_;

	// Common Data...
	nsaddr_t myaddr_;	// My-own-address
	int radius_;   		// Zone-Radius
	int tx_; 		// total pkts transmitted by agent
	int rx_; 		// total pkts received by agent
	int queryID_;		// IERP query IDs

	// General Objects...
	SendBuffer sendBuf_;
	PacketUtil pktUtil_;
	NDPAgent ndpAgt_;
	IARPAgent iarpAgt_;
	IERPAgent ierpAgt_;

	// Constructors...
	ZRPAgent(); 		// Default Constructor - (1)
	ZRPAgent(nsaddr_t id);	// Parameterized Constructor - (2)

	// Methods...
		// 1. General...
		void startUp();
		int  command (int argc, const char*const* argv);
		void recv(Packet * p, Handler *);
		void route_pkt(Packet* p, nsaddr_t dest);
		void route_SendBuffer_pkt();
		void sendPacketUsingIARPRoute(Packet *p, nsaddr_t dest, Time delay);
		int  initialized() { return 1 && target_; }
		void print_tables();

		// 2. Mac Failed...
		void mac_failed(Packet *p);
	
		// 3. Methods for Packet Handling
		Packet* Myallocpkt() {
			Packet *p = allocpkt(); 	// fresh new packet
			return (p);
		}
		void Mydrop(Packet *p, const char *s) {
			 drop(p, s);
		}
		void XmitPacket(Packet *p, Time randomJitter) {
			Scheduler & s = Scheduler::instance();
		  	s.schedule(target_, p, randomJitter);
		}
};
```

---

#### 5.4.1 ZRPAgent类内方法startUp()

```cpp
/* 3. Start Up Function(s). */
void 
ZRPAgent::startUp() {
	// Clear the send buffer...
	sendBuf_.freeList();
	// Start all Sub-Agents...
	ndpAgt_.startUp();
	iarpAgt_.startUp();
	ierpAgt_.startUp();
}
```

---

#### 5.4.2 ZRPAgent类内方法command()

```cpp
/* 4. TCL interface. */
int 
ZRPAgent::command (int argc, const char*const* argv) {
	Time now = Scheduler::instance().clock(); 		// get the time
	// [First set: if argc == 2]
	if (argc == 2) { // No argument from TCL
		if (strcmp (argv[1], "start") == 0) { // Init ZRP Agent
			startUp();
      			return (TCL_OK);
    		} else if (strcasecmp (argv[1], "ll-queue") == 0) { // who is my ll	//[There is no argv[2] here ??]
      			if (!(ll_queue = (PriQueue *) TclObject::lookup (argv[2]))) {
				fprintf (stderr, "ZRP_Agent: ll-queue lookup "
		 		"of %s failed\n", argv[2]);
				return TCL_ERROR;
      			}
      			return TCL_OK; 
    		}
    	// [Second set: if argc == 3]
	} else if (argc == 3)   { // One argument from TCL		// [Parameter 1 - RADIUS]
    		if (strcasecmp (argv[1], "radius") == 0) {		// change the radius, takes (int) num hops
      			int temp;
      			temp = atoi(argv[2]);
      			if (temp > 0) { // don't change radius unless input is valid value
				printf("_%2d_ [%6.6f] | Radius change from %d to %d ",myaddr_, now,
		       		radius_, temp);
				print_tables(); printf("\n");
				radius_ = temp;
      			}
      			return TCL_OK;
    		}							// [Parameter 2 - MIN_BEACON_PERIOD]
    		if (strcasecmp (argv[1], "beacon_period") == 0) {	// change beacon period, takes (int) number of secs
      			int temp;
      			temp = atoi(argv[2]);
      			if (temp > 0) { // don't change unless input is valid value
				printf("_%2d_ [%6.6f] | Beacon period change from %d to %d ",
	       			myaddr_,now, ndpAgt_.BeaconTransmitTimer_.beacon_period_, temp);
				print_tables(); printf("\n");
				ndpAgt_.BeaconTransmitTimer_.beacon_period_ = temp;
      			}
      			return TCL_OK;
    		}/*
    		// change beacon period jitter, takes (int) number of secs
    		if (strcasecmp (argv[1], "beacon_period_jitter") == 0) { 
      			int temp;
      			temp = atoi(argv[2]);
      			if (temp > 0) { // don't change unless input is valid value
				printf("_%2d_ [%6.6f] | Beacon period jitter change from %d to %d ",
	       			myaddr_,now, beacon_period_jitter_, temp);
				print_tables();
				printf("\n");
				beacon_period_jitter_ = temp;
      			}
      			return TCL_OK;
    		}
    		// change neighbor timeout, takes (int) number of secs    
    		if (strcasecmp (argv[1], "neighbor_timeout") == 0) {
      			int temp;
      			temp = atoi(argv[2]);
      			if (temp > 0) { // don't change unless input is valid value
				printf("_%2d_ [%6.6f] | Neighbor timeout change from %d to %d ",
	       			myaddr_,now, neighbor_timeout_, temp);
				print_tables();
				printf("\n");
				neighbor_timeout_ = temp;
      			}
      			return TCL_OK;
    		}
    		// change neighbor ack timeout, takes (int) number of secs    
    		if (strcasecmp (argv[1], "neighbor_ack_timeout") == 0) {
      			int temp;
      			temp = atoi(argv[2]);
      			if (temp > 0) { // don't change unless input is valid value
				printf("_%2d_ [%6.6f] | Neighbor timeout change from %d to %d ",
	       			myaddr_,now, neighbor_ack_timeout_, temp);
				print_tables();
				printf("\n");
				neighbor_ack_timeout_ = temp;
      			}
      			return TCL_OK;
    		}*/
    		// resets myid_ to correct address
    		if (strcasecmp (argv[1], "addr") == 0) {
      			//char str;
      			myaddr_ = Address::instance().str2addr(argv[2]);
      			delete myid_; // remove old string @ myid_, avoid memory leak
      			myid_ = Address::instance().print_nodeaddr(myaddr_);
      			return TCL_OK;
    		}
    		// Error if we cannot find TclObject
    		TclObject *obj;
    		if ((obj = TclObject::lookup (argv[2])) == 0) {
      			fprintf (stderr, "%s: %s lookup of %s failed\n", __FILE__, argv[1],
	       		argv[2]);
      			return TCL_ERROR;
    		}
    		
		// Returns pointer to a trace target 
    		if (strcasecmp (argv[1], "tracetarget") == 0)  {
      			tracetarget = (Trace *) obj;
      			return TCL_OK;
    		}
    		// Returns pointer to our node
    		else if (strcasecmp (argv[1], "node") == 0) {
      			node_ = (MobileNode*) obj;
      			return TCL_OK;
    		}
    		// Returns pointer to port-demux
    		else if (strcasecmp (argv[1], "port-dmux") == 0) {
      			port_dmux_ = (NsObject *) obj;
			return TCL_OK;
    		}
  	}  // if argc == 3
  	return (Agent::command (argc, argv));
}
```

---

#### 5.4.3 ZRPAgent类内方法recv()

```cpp
/* 5. Receiving Packets. */
void 
ZRPAgent::recv(Packet * p, Handler *h) {
	hdr_ip *hdrip = HDR_IP(p);
  	hdr_cmn *hdrc = HDR_CMN(p);
  	hdr_zrp *hdrz = HDR_ZRP(p);
  	int src = Address::instance().get_nodeaddr(hdrip->saddr());
	
	rx_++;		// just received a new packet
	// [Case-1]: It's not a valid ZRP packet
  	if (hdrc->ptype() != PT_ZRP) {
		// [SubCase-1]: A packet from above Layer
		if(src == myaddr_ && hdrc->num_forwards() == 0) {
			hdrc->size() += IP_HDR_LEN;     // Add the IP Header size
    			hdrip->ttl() = IERP_TTL;
			route_pkt(p, hdrip->daddr());	// Route the Packet
		// [SubCase-2]: receiving a pkt I sent, prob a routing loop
		} else if (src == myaddr_) {
			pktUtil_.pkt_drop(p);
    			return;
		// [SubCase-3]: forwarding a non-ZRP packet from another node
  		} else {
			printf("TEMPLOG--AT(%d) I DONOT KNOW WHAT TO DO WITH THIS PACKET--\n", myaddr_);
			exit(0);
    			if(--hdrip->ttl() == 0) {	// Check the TTL.  If it is zero, then discard.
				pktUtil_.pkt_drop(p);
      				return;
    			}
			// WHAT do I do with non-ZRP packets rcvd from another node?
    			// just route_pkt it like any other? then this else clause is superfluous
  		}
	// [Case-2]: It's a valid ZRP packet
	} else if (hdrc->ptype() == PT_ZRP) {
	  	//[Check]: assert we're talking from network layer(rtagent) to network layer(rtagent)
	  	assert(hdrip->sport() == RT_PORT); assert(hdrip->dport() == RT_PORT); assert(hdrc->ptype() == PT_ZRP);
		switch(hdrz->zrptype_) {
			case NDP_BEACON:
				ndpAgt_.recv_NDP_BEACON(p);
			break;
			case NDP_BEACON_ACK:
				ndpAgt_.recv_NDP_BEACON_ACK(p);
			break;
			case IARP_UPDATE:
				iarpAgt_.recv_IARP_UPDATE(p);
			break;	
			case IARP_DATA:
				iarpAgt_.recv_IARP_DATA(p);
			break;
			case IERP_REQUEST:
				if(ierpAgt_.brpXmitPolicy_ == BRP_MULTICAST) {
					//ierpAgt_.recv_IERP_ROUTE_REQUEST_MC(p);
				} else {
					ierpAgt_.recv_IERP_ROUTE_REQUEST_UNI(p);
				}
			break;
			case IERP_REPLY:
				ierpAgt_.recv_IERP_ROUTE_REPLY(p);
			break;
			case IERP_ROUTE_ERROR:
				ierpAgt_.recv_IERP_ROUTE_ERROR(p);
			break;
			case IERP_DATA:
				ierpAgt_.recv_IERP_DATA(p);
			break;
    			default:
      				// error, unknown zrptype, drop 
      				fprintf(stderr, "Invalid ZRP type (%x)\n", hdrz->zrptype_);
      				exit(1);
		}
	}	
}
```

---

#### 5.4.4 ZRPAgent类内方法route_pkt()

```cpp
/* 6. Route the Packet. */
void 
ZRPAgent::route_pkt(Packet* p, nsaddr_t dest) {
  	hdr_cmn *hdrc = HDR_CMN(p);
  	hdr_ip *hdrip = HDR_IP(p);
  	hdr_zrp *hdrz = HDR_ZRP(p);
	
	// [Task-1]: Check if IARP route is Available OR not...
	InnerRoute *handleToInnerRoute = NULL;
	int foundInnerRoute;
	foundInnerRoute = iarpAgt_.irLst_.findRoute(dest, &handleToInnerRoute);
	if(foundInnerRoute == TRUE) {				// IARP route is Available 
		sendPacketUsingIARPRoute(p, dest, 0.00);
		return;
	}

	// [Task-2]: Both IARP & IERP routes are not available.
	Time now = Scheduler::instance().clock(); // get the time
	SentQuery *handleToFoundQuery = NULL;
	int foundQueryFlag;
	// [SubTask-1]: Initiate Route-Discovery if needed.
	foundQueryFlag = ierpAgt_.sqLst_.findQuery(myaddr_, dest, &handleToFoundQuery);
	if(foundQueryFlag == FALSE) { // If No query is generated before
		Packet *pnew;
		pnew = pktUtil_.pkt_create(IERP_REQUEST, dest, IERP_TTL);
		hdr_zrp *hdrznew = HDR_ZRP(pnew);
		hdrznew->queryID_ = queryID_;	queryID_++;	if(queryID_ == 1000000) {queryID_ = 0;}
		hdrznew->routelength_ = 0;	// No route till now
		hdrznew->routeindex_ = 0;	// Route_Index points to the sender(or forwarder) of a query
		hdrznew->src_ = myaddr_;	
		hdrznew->dest_ = dest;		// dest address
		hdrznew->lastbc_ = myaddr_;	// Miss-leading but consistent with 'recv_IERP_ROUTE_REQUEST_XX' call
		// [SubTask-2]: Add this query to Sent-Query-Cache..
		ierpAgt_.sqLst_.addQuery(myaddr_, dest, now+ierpAgt_.queryRetryTime_);
		// [SubTask-3]: Bordercast this query
		if(ierpAgt_.brpXmitPolicy_ == BRP_MULTICAST) {	// This will add entry into cache
			//ierpAgt_.recv_IERP_ROUTE_REQUEST_MC(pnew);	// And Send the ROUTE_REQUEST
		} else {
			ierpAgt_.recv_IERP_ROUTE_REQUEST_UNI(pnew);
		}
	}
	// [SubTask-2]: Insert Packet into the Send Buffer.
	if(sendBuf_.numPackets_ <= DEFAULT_SEND_BUFFER_SIZE) {	// Checking Buffer Overflow [Drop-Tail]
		Time now = Scheduler::instance().clock(); // get the time
		sendBuf_.addPacket(p, dest, now+DEFAULT_PACKET_LIFETIME);
	} else {	
					if(DEBUG) { // [Log: XMIT_CBR_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | XMIT_CBR_DATA | -S %d -D %d | NO_ROUTE_and_SEND_BUF_DROP | ",
					myaddr_, now, hdrip->saddr(), hdrip->daddr());
					print_tables(); printf("\n"); 
					} // [Log: End]
					if(DEBUG) { // [Log: DROP_CBR_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | DROP_CBR_DATA | -S %d -D %d | NO_ROUTE_and_SEND_BUF_DROP | ",
					myaddr_, now, hdrip->saddr(), hdrip->daddr());
					print_tables(); printf("\n"); 
					} // [Log: End]
		drop(p, DROP_RTR_QFULL);// drops the packet
		//assert(p != NULL);
		//Packet::free(p); 	// Not needed anymore
	}
}
```

---

#### 5.4.5 ZRPAgent类内方法route_SendBuffer_pkt()

```cpp
/* 7. Send Packets from the Send_Buffer.*/
void 
ZRPAgent::route_SendBuffer_pkt() {
	// 1. Scan the SendBuffer and send packets for which routes are available.
	if(sendBuf_.numPackets_ != 0) {
		SendBufferEntry *prev, *cur, *toBeDeleted;
		InnerRoute *handleToInnerRoute = NULL;
		int foundInnerRoute;
		// Case-1: Non-head_ case
		prev = sendBuf_.head_; cur = sendBuf_.head_->next_;
		for(int i=0; cur!=NULL; i++) {
			handleToInnerRoute = NULL; foundInnerRoute = FALSE;
			foundInnerRoute = iarpAgt_.irLst_.findRoute(cur->dest_, &handleToInnerRoute);
			if(foundInnerRoute == TRUE) {	// IARP route is Available 
				Time jitter;
				jitter =  Random::uniform(DEFAULT_INTERPKT_JITTER);
				sendPacketUsingIARPRoute(cur->pkt_, cur->dest_, jitter);
				toBeDeleted = cur;
				prev->next_ = cur->next_;
				cur = cur->next_;
				delete toBeDeleted;
				sendBuf_.numPackets_--;
			} else {
				prev = cur;
				cur = cur->next_;
			}
		}
		// Case-2: head_ case
		foundInnerRoute = iarpAgt_.irLst_.findRoute((sendBuf_.head_)->dest_, &handleToInnerRoute);
		if(foundInnerRoute == TRUE) {	// IARP route is Available 
			Time jitter;
			jitter =  Random::uniform(DEFAULT_INTERPKT_JITTER);
			sendPacketUsingIARPRoute((sendBuf_.head_)->pkt_, (sendBuf_.head_)->dest_, jitter);
			toBeDeleted = sendBuf_.head_;
			sendBuf_.head_ = sendBuf_.head_->next_;
			delete toBeDeleted;
			sendBuf_.numPackets_--;
		}
	}
}
```

---

#### 5.4.6 ZRPAgent类内方法sendPacketUsingIARPRoute()

```cpp
/* 8. Send Packet Using IARP route. */
void 
ZRPAgent::sendPacketUsingIARPRoute(Packet *p, nsaddr_t dest, Time delay) {
  	hdr_cmn *hdrc = HDR_CMN(p);
  	hdr_ip *hdrip = HDR_IP(p);
  	hdr_zrp *hdrz = HDR_ZRP(p);

	// [SubTask-1]: Save upper layer info in encapsulated area
	hdrz->enc_daddr_ = hdrip->daddr();
	hdrz->enc_dport_ = hdrip->dport();
	hdrz->enc_ptype_ =  hdrc->ptype();
	// [SubTask-2]: Add the Route
	iarpAgt_.addRouteInPacket(dest, p);
	hdrz->routeindex_ = 0;				// Pointing to My address
	// [SubTask-3]: Change Packet Parameters
	hdrc->direction() = hdr_cmn::DOWN; 		// Common Header
	hdrc->ptype() = PT_ZRP;				// 	''
	hdrc->next_hop() = hdrz->route_[1];		// 	''	[Next Hop]
	hdrc->addr_type_ = NS_AF_NONE;			// 	''
	hdrip->ttl() = hdrz->routelength_;		// IP Header
	hdrip->saddr() = myaddr_;	 		// 	''
	hdrip->daddr() = hdrz->route_[1]; 		// 	''	[Next Hop]
	hdrip->sport() = ROUTER_PORT;	 		// 	''
	hdrip->dport() = ROUTER_PORT;			// 	''
	if(hdrz->routelength_ <= radius_) {
		hdrz->zrptype_ = IARP_DATA;			// ZRP Header
	} else {
		hdrz->zrptype_ = IERP_DATA;			// ZRP Header
	}
	hdrz->src_ = myaddr_;				//	''
	hdrz->dest_ = dest;				//	''
	hdrz->pktsent_ = Scheduler::instance().clock(); //	''
	hdrz->seq_ = pktUtil_.seq_;			//	''
	pktUtil_.inc_seq();
	// [SubTask-4]: Send the Packet
	pktUtil_.pkt_send(p, hdrip->daddr(), 0.00);
					if(DEBUG && hdrz->zrptype_ == IARP_DATA) { // [Log: XMIT_IARP_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | XMIT_CBR_DATA | -S %d -D %d | -SEQ %d -Type IARP | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrz->seq_);
					pktUtil_.pkt_print_route(p); print_tables(); printf("\n"); 
					} // [Log: End]
					if(DEBUG && hdrz->zrptype_ == IARP_DATA) { // [Log: XMIT_IARP_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | XMIT_IARP_DATA | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(), hdrz->seq_);	
					pktUtil_.pkt_print_route(p); print_tables(); printf("\n"); 
					} // [Log: End]
					if(DEBUG && hdrz->zrptype_ == IERP_DATA) { // [Log: XMIT_IARP_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | XMIT_CBR_DATA | -S %d -D %d | -SEQ %d -Type IERP | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrz->seq_);
					pktUtil_.pkt_print_route(p); print_tables(); printf("\n"); 
					} // [Log: End]
					if(DEBUG && hdrz->zrptype_ == IERP_DATA) { // [Log: XMIT_IERP_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | XMIT_IERP_DATA | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(), hdrz->seq_);	
					pktUtil_.pkt_print_route(p); print_tables(); printf("\n"); 
					} // [Log: End]

}

```

---

#### 5.4.7 ZRPAgent类内方法print_tables()

```cpp
/* 9. Printing Tables. */
void 
ZRPAgent::print_tables() {
	ndpAgt_.print_tables();
	iarpAgt_.print_tables();
	ierpAgt_.print_tables();
}

```

---

#### 5.4.8 ZRPAgent类内方法mac_failed()

```cpp
/* 10. As a part of Route-Maintainance for IERP. */
void 
ZRPAgent::mac_failed(Packet *p) {
	hdr_ip *hdrip = HDR_IP(p);
	hdr_cmn *hdrc = HDR_CMN(p);
	hdr_zrp *hdrz = HDR_ZRP(p);
	int nexthop;

	switch (hdrz->zrptype_) {
		case IERP_DATA:
					if(DEBUG) { // [Log: DROP_CBR_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | DROP_CBR_DATA | -S %d -D %d | MAC_FAILED -SEQ %d -Type IERP | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrz->seq_);
					print_tables(); printf("\n"); 
					} // [Log: End]	
			nexthop = hdrz->route_[hdrz->routeindex_+1];	// This pkt was unable to reach this NEXT HOP...
			// Use the same packet and Xmit back it towards Sender
			hdrz->zrptype_ = IERP_ROUTE_ERROR;
			pktUtil_.pkt_add_LSU_space(p, 1); 	// Add the broken [ONLY one]link info...
		  	hdrz->links_[0].src_ = myaddr_;		// Currently this Info is not being used...	
  			hdrz->links_[0].dest_ = nexthop;	// But may be worth sending.. so can be used...	
		  	hdrz->links_[0].isUp_ = FALSE;

			// [Case-1]: Failed at source...
			if(hdrz->src_ == myaddr_) {	
				ierpAgt_.recv_IERP_ROUTE_ERROR(p);
				return;
			}

			// [Case-2]: Failes along the route...
			assert(hdrz->routeindex_-1 >= 0);
			hdrc->direction() = hdr_cmn::DOWN;
			hdrip->saddr() = myaddr_;
			hdrip->daddr() = hdrz->route_[hdrz->routeindex_-1];
			pktUtil_.pkt_send(p, hdrip->daddr(), 0.00);
					if(DEBUG) { // [Log: XMIT_IERP_ERROR]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | XMIT_IERP_ERROR | -S %d -D %d | -IS %d -ID %d | -SEQ %d | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrip->saddr(), hdrip->daddr(), hdrz->seq_);
					pktUtil_.pkt_print_route(p); print_tables(); printf("\n");
					} // [Log: End]

			if(IERP_ERROR_SNOOP) {
				// Remove the broken link info from topology table
				ierpAgt_.removeLinkStateFromBrokenRoute(myaddr_, nexthop);
			}
		break;

		case IARP_DATA:
			// Currently, just logging the event, Not taking any action for this event.
					if(DEBUG) { // [Log: DROP_CBR_DATA]
					Time now = Scheduler::instance().clock(); // get the time
					printf("\n_%d_ [%6.6f] | DROP_CBR_DATA | -S %d -D %d | -SEQ %d MAC_FAILED -Type IARP | ",
					myaddr_, now, hdrz->src_, hdrz->dest_, hdrz->seq_);
					print_tables(); printf("\n"); 
					} // [Log: End]	
		break;

		default:
		break;
	}
}

```

---

