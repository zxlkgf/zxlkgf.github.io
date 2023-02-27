# 集合类


## 2.集合类

### 2.1 Java中有哪些容器(集合类)?

```context
Java中的集合类主要由Collection和Map这两个接口派生而出，其中Collection接口又派生出
三个子接口，分别是Set，List和Queue。所有的Java集合类，都是Set，List，Queue，Map
这四个接口的实现类，这四个接口将集合分成四大类，其中
1.Set代表无序，元素不可重复的集合
2.List代表有序，元素可以重复的集合
3.Queue代表先进先出(FIFO)的队列
4.Map代表具有映射关系的集合
这些接口拥有众多的实现类，其中最常用的实现类有HashSet，TreeSet，ArrayList，
LinkedList，ArrayDeque，HashMap，TreeMap
```

---

### 2.2 Java中的容器，线程安全和线程不安全的分别有哪些？

```context
java.util包下的集合类大部分都是线程不安全的，例如我们常用的HashSet，TreeSet，ArrayList
LinkedList,ArrayDeque,HashMap,TreeMap,这些都是线程不安全的集合类，但是他们的优点
就是性能好。如果需要使用线程安全的集合类，则可以使用Collection工具类提供的synchronizedXXX
方法，将这些集合类包装成线程安全的集合类
---
java.util包下也有线程安全的集合类，例如vector，Hashtable。这些集合类都是比较古老的API
虽然实现了线程安全，但是性能差。所以即便是需要使用线程安全的集合类，也建议将线程不安全的
集合包装成线程安全的集合类的方式，而不是使用古老的API
---
从Java5，java在java.util.concurrent包下提供了大量支持高速并发访问的集合类，他们既能
包装良好的访问性能，也能包装线程安全。这些集合类可以分为两部分，他们的特征如下：
1.以Concurrent开头的集合类
代表了支持并发访问的集合，它们可以支持多个线程并发写入访问，这些写入线程的所有操作都是
线程安全的，但读取操作不必锁定。因其采用了更复杂的算法来保证永远不会锁定整个集合，因此
在并发写入时有较好的性能。
2.以CopyOnWrite开头的集合类
采用了复制底层数组的方式来实现写操作。当线程对此类集合执行读写操作时，线程将会直接读取
集合本身，无需加锁与阻塞。当线程对此类集合执行写入操作时，集合会在底层复制一份新的数组
接下来堆新的数组执行写入操作。由于对集合的写入操作都是对数组的副本执行操作，因此它是
线程安全的。
```

---

### 2.3 Map接口有哪些实现类

```context
Map接口比较常用的有，HashMap，LinkedHashMap，TreeMap，ConcurrentHashMap
对于不需要排序的场合，优先使用HashMap，因为它是性能最好的。需要保证线程安全，则可以
使用ConcurrentHashMap，它的性能好于Hashtable，因为它在put时采用分段锁/CAS的加锁
机制，而不是像Hashtable那样，无论是put还是get都做同步处理。
```

---

### 2.4 描述一下Map put的过程

HashMap是最经典的Map实现，下面以它的视角介绍put的过程：

1.第一次扩容:

先判断数组否为空，或者数组长度是否小于0，如果满足以上条件则进行第一次扩容(resize)

2.计算索引:

通过table[n-1&hash]来计算键值对在数组中的索引

3.插入数据

如果当前位置元素为空，则直接插入数据

如果当前位置元素不为空，且key已存在，则直接覆盖value

如果当前位置元素不为空，且key不存在，则将数组加入到链表末尾(如果节点为树节点，就调用树的put方法)

若链表长度大于8，则将链表转为红黑树，并将数据插入。

4.再次扩容

如果数组中的元素个数超多threshold，则再次调用resize()进行扩容

---

### 2.5 如何得到一个线程安全的Map

1.使用Collections工具类，将线程不安全的Map包装成线程安全的Map;

2.使用java.util.concurrent包下的Map，如ConcurrentHashMap；

3.不建议使用Hashtable，线程安全，但是性能较差

---

### 2.6 HashMap有什么特点

1.HashMap是线程不安全

2.HashMap可以使用null作为key或者value

---

### 2.7 JDK7和JDK8中的HashMap有什么区别

JDK7 中的HashMap，基于数组+链表来实现的，它的底层维护一个Entry数组，它会根据计算的hashCode将对应的键值对存储到该数组中，一旦发生哈希冲突，那么就会将对应的键值对放到对应的已有元素的后面，一次便形成了一个链表式的存储结构。

JDK7中的HashMap的实现方案有一个明显缺点，就是Hash冲突越来越严重的情况下，桶上形成的链表会越来越多，这会导致查询时效率低下，其时间复杂度为O(N)。

JDK8中的HashMap，是基于数组+链表+红黑树实现的，底层维护一个Node数组，当链表存储的数据个数大于8时，将原本的链表转变为红黑树，这样做主要是减少查询时间，将原本链表的O(N)时间，转化为红黑树的O(logN)。提高查询性能.。

---

### 2.8 介绍一个HashMap底层实现原理

它基于hash算法，通过put方法和get方法存储和获取对象。

存储对象时，我们将K/V传给put方法时，它调用K的hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Facotr则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。

如果发生碰撞的时候，HashMap通过链表将产生碰撞冲突的元素组织起来。在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

---

### 2.9 介绍一下HashMap的扩容机制

1. 数组的初始容量为16，而容量是以2的次方扩充的，一是为了提高性能使用足够大的数组，二是为了能使用位运算代替取模预算(据说提升了5~8倍)。

2. 数组是否需要扩充是通过负载因子判断的，如果当前元素个数为数组容量的0.75时，就会扩充数组。这个0.75就是默认的负载因子，可由构造器传入。我们也可以设置大于1的负载因子，这样数组就不会扩充，牺牲性能，节省内存。

3. 为了解决碰撞，数组中的元素是单向链表类型。当链表长度到达一个阈值时（7或8），会将链表转换成红黑树提高性能。而当链表长度缩小到另一个阈值时（6），又会将红黑树转换回单向链表提高性能。

4. 对于第三点补充说明，检查链表长度转换成红黑树之前，还会先检测当前数组数组是否到达一个阈值（64），如果没有到达这个容量，会放弃转换，先去扩充数组。所以上面也说了链表长度的阈值是7或8，因为会有一次放弃转换的操作。

---

### 2.10 HashMap的循环链表是如何产生的?

在JDK7中，多线程情况下，当重新调整HashMap大小的时候，就会存在条件竞争，因为如果有两个线程都发现HashMap需要重新调整大小，则他们都会同时调整大小。在调整大小的过程中，存储在链表的元素的次序会反过来，因为移动到新的bucket位置的时候，链表采取的是头插法移动元素，这是为了防止尾部遍历。但是这也导致了死循环的产生。

---

### 2.11 HashMap为什么用红黑树而不用B树

B/B+树多用于外存上，B/B+也被称为磁盘友好的数据结构。

HashMap本来就是数组+链表的结构，链表由于其查找慢的特点，所以需要被查找效率高的树结构替换。如果用B/B+树，在数据量不是很多的话，数据会挤在一个节点里，遍历效率会退化成链表。

---

### 2.12 HashMap为什么线程不安全

JDK7下的HashMap在并发执行put操作的时候，可能会导致循环链表，从而导致死循环。

---

### 2.13 HashMap如何实现线程安全。

1.直接使用HashTable

2.使用ConcurrentHashMap

3.使用Collections将HashMap包装成线程安全的Map

---

### 2.14 HashMap是如何解决哈希冲突的

为了解决碰撞，数组中的元素是单向链表类型，当链表长度达到一个阈值时，会将链表转换成红黑树提高性能。而当链表长度缩小到一个阈值时，又会将红黑树退化成链表提高性能.

---

### 2.15 说说HashMap和HashTable的区别

1. Hashtable是一个线程安全的Map实现，但HashMap是线程不安全的实现，所以HashMap比Hashtable的性能高一点。

2. Hashtable不允许使用null作为key和value，如果试图把null值放进Hashtable中，将会引发空指针异常，但HashMap可以使用null作为key或value。

---

### 2.16 HashMap与ConcurrentHashMap有什么区别？

HashMap是非线程安全的，这意味着不应该在多线程中对这些Map进行修改操作，否则会产生数据不一致的问题，甚至还会因为并发插入元素而导致链表成环，这样在查找时就会发生死循环，影响到整个应用程序。

Collections工具类可以将一个Map转换成线程安全的实现，其实也就是通过一个包装类，然后把所有功能都委托给传入的Map，而包装类是基于synchronized关键字来保证线程安全的（Hashtable也是基于synchronized关键字），底层使用的是互斥锁，性能与吞吐量比较低。

ConcurrentHashMap的实现细节远没有这么简单，因此性能也要高上许多。它没有使用一个全局锁来锁住自己，而是采用了减少锁粒度的方法，尽量减少因为竞争锁而导致的阻塞与冲突，而且ConcurrentHashMap的检索操作是不需要锁的。

---

### 2.17 介绍一下ConcurrentHashMap是怎么实现的

JDK1.7中的实现:

在jdk1.7中，ConcurrentHashMap是由segment数据结构和HashEntry数据结构构成。采取分段锁来保证安全性。Segment是ReentrantLock重入锁，在ConcurrentHashMap中扮演锁的角色，HashEntru则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，一个segment数组里包含一个HashEntry数组，Segment的结构和HashMap相似，是一个数组和链表结构。

JDK1.8中的实现:

JDK1.8的实现已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来表现，并发控制使用了Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到segment的数据结构，但是已经简化了属性，只是为了兼容旧版本。

---

### 2.18 ConcurrentHashMap是怎么分段分组的？

get操作：

Segment的get操作实现非常简单和高效，先经过一次再散列，然后使用这个散列值通过散列运算定位到 Segment，再通过散列算法定位到元素。get操作的高效之处在于整个get过程都不需要加锁，除非读到空的值才会加锁重读。原因就是将使用的共享变量定义成 volatile 类型。

put操作：

当执行put操作时，会经历两个步骤：

1. 判断是否需要扩容；

2. 定位到添加元素的位置，将其放入 HashEntry 数组中。

插入过程会进行第一次 key 的 hash 来定位 Segment 的位置，如果该 Segment 还没有初始化，即通过 CAS 操作进行赋值，然后进行第二次 hash 操作，找到相应的 HashEntry 的位置，这里会利用继承过来的锁的特性，在将数据插入指定的 HashEntry 位置时（尾插法），会通过继承 ReentrantLock 的 tryLock() 方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用 tryLock() 方法去获取锁，超过指定次数就挂起，等待唤醒。

---

### 2.19 说一说你对LinkedHashMap的理解

LinkedHashMap使用双向链表来维护key-value对的顺序（其实只需要考虑key的顺序），该链表负责维护Map的迭代顺序，迭代顺序与key-value对的插入顺序保持一致。

LinkedHashMap可以避免对HashMap、Hashtable里的key-value对进行排序（只要插入key-value对时保持顺序即可），同时又可避免使用TreeMap所增加的成本。

LinkedHashMap需要维护元素的插入顺序，因此性能略低于HashMap的性能。但因为它以链表来维护内部顺序，所以在迭代访问Map里的全部元素时将有较好的性能。

---

### 2.20 请介绍LinkedHashMap的底层原理

LinkedHashMap继承于HashMap，它在HashMap的基础上，通过维护一条双向链表，解决了HashMap不能随时保持遍历顺序和插入顺序一致的问题。在实现上，LinkedHashMap很多方法直接继承自HashMap，仅为维护双向链表重写了部分方法。

---

### 2.21 请介绍TreeMap的底层原理

TreeMap基于红黑树实现.映射根据其键的自然顺序进行排序，或者根据创建映射时提供的Comparator进行排序，具体取决于使用的构造方法。TreeMap的时间复杂度是O(logN)。

TreeMap包含几个重要的成员变量:root，size，comparator。其中root是红黑树的根节点。它是Entry类型，Entry是红黑树的节点，它包含了红黑树的6个基本组成，key，value，left，right，parent和color。Entry节点根据key排序，包含的内容是value。Entry中的key大小是根据comparator进行比较的。size是红黑树的节点个数。

---

### 2.22 Map和Set有什么区别

Map代表具有映射关系的集合，其所有的Key是一个Set集合，即key无序且不能重复

Set代表无序，元素不可重复的集合

---

### 2.23 List和Set有什么区别

Set代表无序，元素不可重复的集合

List代表有序，元素可以重复的集合

---

### 2.24 ArrayList和LinkedList的区别

1. ArrayList的实现是基于数组，LinkedList的实现是基于双向链表；

2. 对于随机访问ArrayList要优于LinkedList，ArrayList可以根据下标以O(1)时间复杂度对元素进行随机访问，而LinkedList的每一个元素都依靠地址指针和它后一个元素连接在一起，查找某个元素的时间复杂度是O(N)；

3. 对于插入和删除操作，LinkedList要优于ArrayList，因为当元素被添加到LinkedList任意位置的时候，不需要像ArrayList那样重新计算大小或者是更新索引；

4. LinkedList比ArrayList更占内存，因为LinkedList的节点除了存储数据，还存储了两个引用，一个指向前一个元素，一个指向后一个元素。

---

### 2.25 有哪些线程安全的List类

1. Vector
   
   Vector是比较古老的API，虽然保证了线程安全，但是由于效率低一般不建议使用。

2. Collections.SynchronizedList
   
   SynchronizedList是Collections的内部类，Collections提供了synchronizedList方法，可以将一个线程不安全的List包装成线程安全的List，即SynchronizedList。它比Vector有更好的扩展性和兼容性，但是它所有的方法都带有同步锁，也不是性能最优的List。

3. CopyOnWriteArrayList
   
   CopyOnWriteArrayList是Java 1.5在java.util.concurrent包下增加的类，它采用复制底层数组的方式来实现写操作。当线程对此类集合执行读取操作时，线程将会直接读取集合本身，无须加锁与阻塞。当线程对此类集合执行写入操作时，集合会在底层复制一份新的数组，接下来对新的数组执行写入操作。由于对集合的写入操作都是对数组的副本执行操作，因此它是线程安全的。在所有线程安全的List中，它是性能最优的方案。

---

### 2.26 介绍一下ArrayList的数据结构？

**参考答案**

ArrayList的底层是用数组来实现的，默认第一次插入元素时创建大小为10的数组，超出限制时会增加50%的容量，并且数据以 System.arraycopy() 复制到新的数组，因此最好能给出数组大小的预估值。

按数组下标访问元素的性能很高，这是数组的基本优势。直接在数组末尾加入元素的性能也高，但如果按下标插入、删除元素，则要用 System.arraycopy() 来移动部分受影响的元素，性能就变差了，这是基本劣势。

---

### 2.27 谈一谈CopyOnWriteArrayList

CopyOnWriteArrayList是Java并发包里提供的并发类，简单来说它就是一个线程安全且读操作无锁的ArrayList。正如其名，在写操作时会复制一份新的List，在新的List上完成写操作，然后再将原引用指向新的List。这样就保证了线程安全。

CopyOnWriteArrayList允许线程并发访问读操作，这个时候是没有加锁限制的，性能较高。而写操作的时候，则首先将容器复制一份，然后在新的副本上执行写操作，这个时候，写操作是上锁的。结束之后再将原容器的引用指向新容器。注意，在上锁执行写操作的过程中。如果有需要读操作，则会作用在原容器上，因此上锁的写操作不会影响到并发访问的新操作。

优点 ：读操作性能高，因此无需任何同步措施，比较适用于读多写少的并发场景。在遍历传统的List时，若中途有别的线程对其进行修改，则会抛出ConcurrentModificationException异常。而CopyOnWriteArrayList由于其读写分离的思想，遍历和修改操作作用于不同的容器，所以在使用迭代器进行遍历时候，就不会抛出ConcurrentModificationException异常了。

缺点：内存占用问题大，数据量大的时候，对内存压力大，可能会频繁引起GC。二是无法保证实时性，Vector对于读写操作均加锁同步，可以保证读和写的一致性。而CopyOnWriteArrayList由于其实现策略的原因，写和读分别作用于新老不同的List上，在执行写操作的过程，读不会阻塞，而是作用在老容器的数据。

---

### 2.28 说一说TreeSet和HashSet的区别

HashSet，TreeSet中的元素是不可重复的，并且他们都是线程不安全的，二者的区别是:

1.HashSet中的元素可以是Null，但是TreeSet中的元素不能是Null

2.HashSet不能保证元素的排列顺序，而TreeSet支持自然排序，定制排序两种排序方式

3.HashSet底层是采用Hash表实现的，而TreeSeet底层采用的是红黑树实现的

---

### 2.29 说一说HashSet的底层结构

HashSet是基于HashMap实现的，默认构造函数是构建一个初始容量为16，负载因子为0.75的HashMap。它封装了一个HashMap对象存储所有的集合元素，所有放入HashSet中的集合元素实际上有HashMap的Key存储，而HashMap的value则存储一个PRESENT，他是一个静态的Object对象

---

