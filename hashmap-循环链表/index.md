# HashMap 循环链表


# 1.说一下HashMap循环链表产生的原理

---

首先我们先了解一下HashMap产生循环链表是出现在JDK1.7版本。

其次产生场景是并发状况下出现死循环

产生原因为:多线程同时put时，如果同时操作resize(),可能会导致链表循环，进而使得后面的get产生死循环。

那么下面我们来看看与其相关联的代码

---

## 1.1 resize()函数

该函数是数组扩容函数，主要功能就是创建扩容后的数组，并调用transfer函数，将旧数组的元素迁移到新数组中。

```java
/*
* HashMap数组扩容函数
* @param:newCapacity 指定新生成数组的大小
*/
void resize(int newCapacity) {
    //存储旧数组的临时变量
    Entry[] oldTable = table;
    //旧数组的大小
    int oldCapacity = oldTable.length;
    //如果旧数组的大小已经达到了2^30，则将扩容阈值修改为2^31-1
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    // 根据新的数组长度，创建数组
    Entry[] newTable = new Entry[newCapacity];
    // 转移数组数据 
    // 调用initHashSeedAsNeeded方法，根据新数组的长度判断是否会hashSeed重新赋值
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    //table指向新的数组
    table = newTable;
    //计算新的扩容阈值
    threshold = (int)(newCapacity * loadFactor);
}
```

---

## 1.2 tansfer()函数

transfer逻辑其实也简单，遍历旧数组，将旧数组元素通过头插法的方式，迁移到新数组的对应位置问题出就出在头插法。

```java
// transfer方法
void transfer(Entry[] newTable, boolean rehash) {
    // 新数组长度
    int newCapacity = newTable.length;
    // 遍历原数组的Entry
    for (Entry<K,V> e : table) {
        // Entry不为null
        while(null != e) {
            // 先找到链表中下一个要转移的Entry
            Entry<K,V> next = e.next;
            // 如果hashSeed变了，重新进行hash计算
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            // 重新计算数组下标，结果分两种：1.与原下标一直 2.原下标+本次扩容了多长
            int i = indexFor(e.hash, newCapacity);
            /*原数组统一链表中的数据转移到新数组中时，所处链表顺序颠倒，因此多线程的情况下可能会出现环形链表问题*/
            // Entry的next指向新数组的链表头
            e.next = newTable[i];
            // Entry放入新数组的链表中
            newTable[i] = e;
            // 下一次进行操作的就是原数组链表的下一个
            e = next;
        }
    }
}
```

---

## 1.3 indexFor() 函数

```java
static int indexFor(int h, int length) {
    // 计算数组下标位置，数组长度必须为2的n次方
    return h & (length-1);
}
```

---

# 2. 例子

1.我们单纯的将indexFor中的计算数组的方法置换为key的值mod数组大小

2.我们的初始HashMap的size为2，阈值为1(2*0.75=1)，其中有key = 3, 7, 5，在mod 2以后都冲突在table[1]这里了.

3.那么接下来，HashMap被resize()函数扩容到4，并经过transfer()函数得到新的HashMap

初始table

| 0   | null   |               |               |               |      |
| --- | ------ | ------------- | ------------- | ------------- | ---- |
| 1   | -----> | key:3,value:A | key:7,value:B | key:5,value:C | null |

---

正常的resize()之后的rehash

| 0   | null          |               |     |
| --- | ------------- | ------------- | --- |
| 1   | key:5,value:C |               |     |
| 2   | null          |               |     |
| 3   | key:7,value:B | key:3,value:A |     |

---

但是如果有两个线程，都执行put操作，并都对该初始table进行resize()操作之后，当进行到tansfer()时将有一个线程进入阻塞状态。这时候也就意味着有一个线程所拿取的e以及next，将指向线程二已经完成的table中的元素。

再观察下述transfer()中的头插法代码

```java
    for (Entry<K,V> e : table) {
        // Entry不为null
        while(null != e) {
            // 先找到链表中下一个要转移的Entry
            Entry<K,V> next = e.next;
            // 如果hashSeed变了，重新进行hash计算
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            // 重新计算数组下标，结果分两种：1.与原下标一直 2.原下标+本次扩容了多长
            int i = indexFor(e.hash, newCapacity);
            /*原数组统一链表中的数据转移到新数组中时，所处链表顺序颠倒，因此多线程的情况下可能会出现环形链表问题*/
            // Entry的next指向新数组的链表头
            e.next = newTable[i];
            // Entry放入新数组的链表中
            newTable[i] = e;
            // 下一次进行操作的就是原数组链表的下一个
            e = next;
        }
    }
```

这将导致之前线程1中的节点再次运行，从而导致出现循环链表。

此时如果调用get方法去查询该hash表内hash值所对应的链表，则会陷入死循环。

