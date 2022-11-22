# HashMap详解


# 1. HashMap源码详解

基于JDK1.8

---

## 1.1HashMap简介

HashMap 主要用来存放键值对，它基于哈希表的 Map 接口实现，是常用的 Java 集合之一，是非线程安全的，且不能保证元素的存储顺序。查询和修改的速度很快，能到到O(1)的平均复杂度。

```context
HashMap可以存储null的key和value
null作为key键只能有一个
null作为value值可以有多个
```

---

# 2 底层数据结构分析

## 2.1 JDK1.8之前

```context
HashMap底层时数组加单向链表，结合在一起使用也就是散列链表。
将key的hash值进行取模运算获取index就是即将存放的元素的位置，然后到对应的链表进行
put和get操作。
```

```java
/**
 * 源码分析1：hash(key)
 * 该函数在JDK 1.7 和 1.8 中的实现不同，但原理一样 = 扰动函数 = 使得根据key生成的哈希码（hash值）分布更加均匀、更具备随机性，避免出现hash值冲突（即指不同key但生成同1个hash值）
 * JDK 1.7 做了9次扰动处理 = 4次位运算 + 5次异或运算
 * JDK 1.8 简化了扰动函数 = 只做了2次扰动 = 1次位运算 + 1次异或运算
 */
// JDK 1.7实现：将 键key 转换成 哈希码（hash值）操作  = 使用hashCode() + 4次位运算 + 5次异或运算（9次扰动）
final int hash(Object k) {
    // 设置了哈希种子
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }
    // Hash种子参与到了key的Hash值计算当中
    h ^= k.hashCode();
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

---

## 2.2 JDK1.8之后

```
HashMap在解决哈希冲突时有了较大的变化,当链表长度大于阈值(默认为8)(将链表转换成红黑树
前会判断,如果当前数组的长度小于 64,那么会选择先进行数组扩容,而不是转换为红黑树)时,将链
表转化为红黑树,以减少搜索时间。
```

```java
/**
 * JDK 1.8实现：将 键key 转换成 哈希码（hash值）操作 = 使用hashCode() + 1次位运算 + 1次异或运算（2次扰动）
 * 1. 取hashCode值： h = key.hashCode() 
 * 2. 高位参与低位的运算：h ^ (h >>> 16)
 */
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    // a. 当key = null时，hash值 = 0，所以HashMap的key 可为null      
    // 注：对比HashTable，HashTable对key直接hashCode（），若key为null时，会抛出异常，所以HashTable的key不可为null
    // b. 当key ≠ null时，则通过先计算出 key的 hashCode()（记为h），然后 对哈希码进行扰动处理： 按位 异或（^） 哈希码自身右移16位后的二进制
}
```

---

## 2.3 HashMap类内成员

首先先来明确几个HashMap的概念：

- HashMap里面数组结构的每一个存储元素的位置被称为桶
- 桶中存放的数据被称为bin（bin这个概念会在HashMap源码中大量出现）

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L；
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 负载因子，当数组的存储比例达到了0.75，数组就会进行两倍扩容
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //链表转成树的阈值，链表中的元素个数大于8，变为红黑树和MIN_TREEIFY_CAPACITY一起决定的
    static final int TREEIFY_THRESHOLD = 8;
    //树转换成链表的阈值 树中的元素小于6，转换为链表
     static final int UNTREEIFY_THRESHOLD = 6;
    //最小转成树的容量，当数组长度达到64转换为红黑树-和TREEIFY_THRESHOLD一起决定的
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，数组长度总是2的幂次倍
    transient Node<k,v>[] table;
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，表示当前HashMap包含的键值对数量。注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器，表示当前HashMap修改次数
    transient int modCount;
    //扩容的阈值，当实际大小为(16 * 0.75 = 12) 阈值时开始扩容 
    int threshold;
    // 加载因子（负载因子）
    final float loadFactor;
}
```

TREEIFY_THRESHOLD: 一个bin是否需要被转化成红黑树的阈值，当bin内元素超过TREEIFY_THRESHOLD 的值的时候，该bin将会被转化为红黑树，以提高查询效率。

UNTREEIFY_THRESHOLD:一个bin是否需要从红黑树转化为链表的阈值，当bin内元素小于UNTREEIFY_THRESHOLD,则该bin将会被转化为链表

MIN_TREEIFY_CAPACITY:最小转化树的容量，即当数组长度达到64转化为红黑树

(MIN_TREEIFY_CAPACITY不太理解)

---

## 2.4 构造方法

```java
//默认无参构造方法
public HashMap() {
    //将默认的负载因子赋值给成员变量loadFactor
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
}

/**
 * 包含另一个“Map”的构造函数，包含另一个Map的映射，如果被映射的Map是一个null会抛出空指针异常。负载因子是默认的
 * 直接传入存储了要添加进HashMap的key-value对的map，来构造HashMap
 */
public HashMap(Map<? extends K, ? extends V> m) {
    //将默认的负载因子赋值给成员变量loadFactor
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    //调用PutMapEntries()来完成HashMap的初始化赋值过程
    putMapEntries(m, false);//下面会分析到这个方法
}

/**
 * 指定“容量大小”的构造函数，直接使用默认负载因子0.75
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

/**
* 构造一个空的HashMap并指定初始容量和负载因子。
* 要注意HashMap源码里面并没有专门的一个属性来存储数组的容量，而是通过threshold来简介限制数组容量的
* 通过将自定义初始化数组容量传入tableSizeFor()方法，计算得出initialCapacity容量大小应该对应的阈值threshold大小
* 这样当数组内元素数大于threshold，就会触发扩容操作，间接限定了数组容量大小
**/
public HashMap(int initialCapacity, float loadFactor) {
    //如果初始容量小于0，抛出非法参数异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    //如果初始容量大于最大的容量也就是2^30,那么就按照最大的初始容量赋值。
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //如果负载因子小于0或者是NaN（float NaN = 0.0f / 0.0f;）也会抛出非法参数异常
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    // 设置重载因子
    this.loadFactor = loadFactor;
    // 调用tableSizeFor方法计算出不小于initialCapacity的最小的2的幂的结果，并赋给成员变量threshold
    // 注意，这里赋给threshold并不是扩容阈值，只是临时赋值。
    //此时HashMap还没有创建数组，当插入数据的时候会判断该HashMap是否已经初始化，那个时候就会执行resize()方法进行一次扩容,就会重新计算一正确的扩容阈值赋值给threshold
    this.threshold = tableSizeFor(initialCapacity);
}
```

---

## 2.5 Node节点源码

```java
/**
 * Basic hash bin node, used for most entries.  (See below for
 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
 */
//继承了Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;//常量:key的hash值
    final K key;//键
    V value;//值
    Node<K,V> next;//下一个节点
    //有参构造方法
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
    //重写hashCode
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //重写equals方法
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

```java
//创建Node节点的方法
// Create a regular (non-tree) node
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}

// For conversion from TreeNodes to plain nodes
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
```

---

## 2.6 TreeNode节点源码

TreeNode是用来实现红黑树的树节点。

红黑树是一个二叉搜索树，他在每个节点增加了一个存储位去记录节点颜色(黑，红)。

红黑树有以下四个性质

1.节点是红色或者黑色

2.根结点一定是黑色

3.所有叶子都是黑色(指的是空节点)(叶子节点的空节点为黑色)

4.每个红色节点的两个子节点都是黑色(从叶子到根的所有路径上不能有两个红色节点)

5.所有根节点到叶子(空节点)节点的路径中，包含的黑色节点数相同

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    /**
     * Returns root of tree containing this node.
     */
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
/*
内部方法省略
*/
}
```

---

## 3.内部方法

---

### 3.1 tableSizeFor()

```java
    /**
     * 计算大于等于参数的第一个2的幂次方
     * 例如，1返回1，3返回4，6返回8
     */
    static final int tableSizeFor(int cap) {
    //容量减1，为了防止原本就是2的幂次方，而导致最后求得的结果为2的幂次方的2倍
        int n = cap - 1;
    //算法部分，
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

---

其中:

n |=n>>>1 即  n = n | n >>> 1;     >>>为无符号右移

---

tableSizeFor()函数:

对任意十进制数转换为2的整数幂，结果是这个数本身的最高有效位的前一位变成1，最高有效位以及其后的位都变为0。

通过上面理论基础，我们可以得出该算法的核心思想是，先将最高有效位以及其后的位都变为1，最后再+1，就进位到前一位变成1，其后所有的满2变0。所以关键是如何将最高有效位后面都变为1。

---

### 3.2 putMapEntries()

下面我们再来看看在传入Map参数的构造方法中调用的putMapEntries()方法。这个方法调用了HashMap的resize()扩容方法和putVal()存入数据方法。

putMapEntries函数会被HashMap的拷贝构造函数public HashMap(Map<? extends K, ? extends V> m)或者Map接口的putAll函数（被HashMap给实现了）调用到。该函数使用的是默认修饰符（default），也就是只有包访问权限，只能被本类或者该包下的类访问到，所以一般情况下用户无法调用。

```java
/**
 * 该方法的作用：将传入的子Map中的全部元素逐个添加到HashMap中
 * @param evict 最初构造此Map时为false，否则为true（中继到afterNodeInsertion方法）。
 */
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    //获取子Map内数据大小
    int s = m.size();
    // 判断
    if (s > 0) {
        // 判断table是否已经初始化  如果table=null一般就是构造函数来调用的putMapEntries，或者构造后还没放过任何元素
        if (table == null) { // pre-size
            // 如果未初始化，则计算HashMap的最小需要的容量（即容量刚好不大于扩容阈值）
            // 这里Map的大小s就被当作HashMap的扩容阈值，然后用传入Map的大小除以负载因子就能得到对应的HashMap的容量大小（当前m的大小 / 负载因子 = HashMap容量）
            // 先不考虑容量必须为2的幂，那么下面括号里会算出来一个容量，使得size刚好不大于阈值。但这样会算出小数来，但作为容量就必须向上取整，所以这里要加1。此时ft可以临时看作HashMap容量大小
            float ft = ((float)s / loadFactor) + 1.0F;
            //比较最大容量与ft，取小值； 到这里t暂时表示HashMap的容量大小。如果是将ft浮点型赋值给t整形，因为前面加了1.0f,这里也就实现了向上取整
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            // 只有在算出来的容量t > 当前暂存的容量(容量可能会暂放到阈值上的,刚使用构造函数构造出来的HashMap并且没有存入元素时，容量大小就会被暂时存在threshold中)时
            // 才会用t计算出新容量，暂时存放到阈值上,在后面触发resize()扩容的时候会对threshold重新计算正确的阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        //如果当前Map已经初始化,且这个map中的元素个数大于扩容的阀值就得扩容
        //这种情况属于预先扩大容量，再put元素
        else if (s > threshold)
            resize();
        //遍历map,将map中的key和value都添加到HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            // 调用HashMap的put方法的具体实现方法putVal来对数据进行存放。该方法的具体细节在后面会进行讲解
            // putVal可能也会触发resize
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

---

### 3.3 put()方法流程

HashMap只提供了put用于添加元素，putval也是使用的默认修饰符，因此只能被本类或者该包下的类访问到，所以putVal 方法只是给 put 方法调用的一个方法，并没有提供给用户使用。

putVal()添加元素的分析如下:

1.如果当前元素为空，直接插入元素

2.如果当前元素非空，就要和插入的key比较，如果key相同就直接覆盖

3.如果当前元素非空，如果key不同，就要将数据插入链表末端

4.如果当前元素非空，如果key不同，判断p是否是一个树节点，如果是就调用((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
 /**
 * Implements Map.put and related methods.
 * 实现了map的put和相关方法
 * @param hash  key的hash值（key的hash高16位+高16位与低16位的异或运算）
 * @param key 键
 * @param value 值  
 * @param onlyIfAbsent onlyIfAbsent为true的时候不要修改已经存在的值，如果onlyIfAbsent为false，当插入的元素已经在HashMap中已经拥有了与其key值和hash值相同的元素，仍然需要把新插入的value值覆盖到旧value上。
 * @param evict evict如果为false表示构造函数调用
 * @return 返回旧的value值（在数组桶或链表或红黑树中找到存在与插入元素key值和hash值相等的元素，就返回这个旧元素的value值），如果没有发现相同key和hash的元素则返回null
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; //用来临时存放数组table的引用
    Node<K,V> p;    //用来临时存放数组tab中的bin的内容
    int n, i;    //n存放Hashmap大小，i表示插入数据的下标
    //如果table为空或者table的长度为0 调用resize()为其扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //i = (n - 1) & hash 表示计算数组的索引赋值给i，即确定元素存放在哪个桶中
    //p = tab[i = (n - 1) & hash] 将计算出的数组索引对应的数据赋值给节点p
    //判断p是否为null，即当前桶没有哈希冲突，则直接把键值对插入空间位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {//说明p不为空，表示这个位置已经有值
        Node<K,V> e;
        K k;
        //p.hash == hash 判断当前bin内内容的hash是否与插入key的hash相等
        //((k = p.key) == key 比较两个key的地址是否相等
        //(key != null && key.equals(k))执行到这里说明key的地址不同
        //判断key不为空，并且判断两个key内容是否相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //两个数据key相同，则将旧数据交给e保存
            e = p;
        //hash值不等，或者key值不等
        //判断p是不是红黑树节点
        else if (p instanceof TreeNode)
            //如果是则调用树的putTreeVal放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //key地址不同内容不同，p不为树节点
        //需要插入链表中
        else {
            //在链表的末尾插入
            for (int binCount = 0; ; ++binCount) {
                //e = p.next 将p的下一个数据赋值给
                //e = p.next) == null)判断是否到达链表的末尾
                if ((e = p.next) == null) {
                    //创建一个新的Node节点插入尾部
                    p.next = newNode(hash, key, value, null);
                    //如果当前链表的长度超过8
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //转换为红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                //继续判断p的下一个节点的数据是否与插入数据的hash相等或者值相等
                //如果相等就退出插入
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //继续循环链表
                p = e;
            }
        }
        /*
            表示在桶中找到key值、hash值与插入元素相等的结点
            也就是说通过上面的操作找到了重复的键，所以这里就是把该键的值变为新的值，
            并返回旧值，这里完成了put方法的修改功能
        */
        if (e != null) { // existing mapping for key
            //记录旧的值
            V oldValue = e.value;
            //onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值:e.value 表示旧值  value表示新值 
                e.value = value;
            // 替换旧值时会调用的方法（默认实现为空）
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改，记录HashMap被修改的次数，主要用于多线程并发时候
    ++modCount;
    //++size当前HashMap存放的键值对进行更新
    //如果大于当前的threshold阈值，则对齐进行扩容
   if (++size > threshold)
        resize();
    // 插入成功时会调用的方法（默认实现为空）
    afterNodeInsertion(evict);
     // 没有找到原有相同key和hash的元素，则直接返回Null
    return null;
}
```

---

#### 3.3.1 内部实现为空的方法

```java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

在putVal()方法中替换旧值和插入成功的时候都调用了上面其中的两个方法，这三个方法是HashMap类中的方法，但是我们查看源码后会发现这三个方法都是空的方法。其实这三个方法是为继承HashMap的LinkedHashMap类服务的。

LinkedHashMap是HashMap的一个子类，它保留插入的顺序，如果需要输出的顺序和输入时的相同，那么就选用 LinkedHashMap

---

#### 3.3.2 treeifyBin()方法

将所有的节点转换成树形节点，并且将链接的链表线索化，即为每个二叉树的节点添加前驱和后继节点，形成线索，构造出双链表结构。再调用treeify()方法构造红黑树结构关系。

```java
/**
 * 将数组指定位置的链表转为红黑树
 * @param tab 数组
 * @param hash 将这个hash值所在的索引上的链表转为红黑
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    // n存储当前table的长度
    // index hash计算得到的索引
    int n, index; 
    // e存储index索引位置的元素
    Node<K,V> e;
    //如果table为空｜｜或者table的大小小于最小树化阈值就调用resize方法，扩容
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    //(n-1)&hash 获取index
    //e = table[index = (n-1) & hash] 获取数组index处的节点，并判断节点是否为空
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        //定义存储树的头节点hd 树的尾节点tl
        TreeNode<K,V> hd = null, tl = null;
        //从e节点开始转换树节点
        do {
            //新建存储e节点转化为树节点的p
            TreeNode<K,V> p = replacementTreeNode(e, null);
            // 如果树头节点tl为空，将p赋值给tl
            if (tl == null)
                hd = p;
            else {
            //否则 将p的前置节点设为tl
            //tl的后置节点设为p
                p.prev = tl;
                tl.next = p;
            }
            //将p赋值给tl
            tl = p;
        } while ((e = e.next) != null);//循环条件e=e.next不为空
        //让桶中的第一个元素即数组中的元素指向新建的红黑树的节点
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}

// For treeifyBin
// 将Node节点转化为树节点的方法
TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    return new TreeNode<>(p.hash, p.key, p.value, next);
}
```

---

### 3.4 Node<K,V>[] resize()

下面看一下 resize 方法，面试时最经常问的hashmap扩容机制就在这个地方，注意newCap = oldCap << 1这句，扩容就在这，扩大两倍。

进行扩容，会伴随着一次重新 hash 分配，并且会遍历 hash 表中所有的元素，是非常耗时的。在编写程序中，要尽量避免 resize。

```java
/**
* Initializes or doubles table size.  If null, allocates in
* accord with initial capacity target held in field threshold.
* Otherwise, because we are using power-of-two expansion, the
* elements from each bin must either stay at same index, or move
* with a power of two offset in the new table.
*
* @return the table
*/
final Node<K,V>[] resize() {
    //oldTab存储当前table的临时变量
    Node<K,V>[] oldTab = table;
    //判断table是否进行初始化 如果为null返回0，否则返回table的长度
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //旧的阈值 如果oldTab为null或者0 oldThr则为0
    //否则为12(16*0.75)
    int oldThr = threshold;
    int newCap, newThr = 0;
    //第一次数组容量肯定是0
    //判断旧数组容量>0
    if (oldCap > 0) {
        //如果数组容量超过最大容量，就直接返回
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //没有超过最大容量,新容量为旧容量的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            //阈值也扩大为两倍
            newThr = oldThr << 1; // double threshold
    }
    //如果原容量不大于0 表示原table为null，就判断旧阈值
    //如果原table为null，代表其只是调用了构造方法，没有真正初始化table，只有插入数据才会扩容
    else if (oldThr > 0) // initial capacity was placed in threshold
        // 将原阈值作为容量赋值给newCap当做newCap的值。由之前的源码分析可知，此时原阈值存储的大小就是调用构造函数时指定的容量大小，所以直接将原阈值赋值给新容量
        newCap = oldThr;
    // 如果原容量不大于0，并且原阈值也不大于0。这种情况说明调用的是无参构造方法，还没有真正初始化HashMap，只有put()数据的时候才会触发扩容操作进而进行初始化
    else {               // zero initial threshold signifies using defaults
        // 则以默认容量作为newCap的值
        newCap = DEFAULT_INITIAL_CAPACITY;//16
        // 以初始容量*默认负载因子的结果作为newThr值
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 经过上面的处理过程，如果newThr值为0，说明上面是进入到了原容量不大于0，旧阈值大于0的判断分支。需要单独给newThr进行赋值
    if (newThr == 0) {
       // 临时阈值 = 新容量 * 负载因子
       float ft = (float)newCap * loadFactor;
       // 设置新的阈值 保证新容量小于最大总量   阈值要小于最大容量，否则阈值就设置为int最大值
       newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    // 将新的阈值newThr赋值给threshold，为新初始化的HashMap来使用
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    //创建新的表
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //将新创建的表赋值给当前类成员
    table = newTab;
    //如果旧表不为空
    //将旧表的元素移动到新表
    if (oldTab != null) {
        // 如果原来的HashMap中有值，则遍历oldTab，取出每一个键值对，存入到新table
        for (int j = 0; j < oldCap; ++j) {
            // 创建一个临时变量e用来指向oldTab中的第j个键值对
            Node<K,V> e;
            // 将oldTab[j]赋值给e并且判断原来table数组中第j个位置是否不为空
            if ((e = oldTab[j]) != null) {
                // 如果不为空，则将oldTab[j]置为null，释放内存
                oldTab[j] = null;
                // 如果e.next = null，说明该位置的数组桶上没有连着额外的数组
                if (e.next == null)
                     // 此时以e.hash&(newCap-1)的结果作为e在newTab中的位置，将e直接放置在新数组的新位置即可
                    newTab[e.hash & (newCap - 1)] = e;
                // 否则说明e的后面连接着链表或者红黑树，判断e的类型是TreeNode还是Node,即链表和红黑树判断
                else if (e instanceof TreeNode)
                    //调用相关方法
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 当前节不是红黑树，不是null，并且还有下一个元素。那么此时为链表
                else { // preserve order
                    /*
                        这里定义了五个Node变量，其中lo和hi是，lower和higher的缩写，也就是高位和低位,
                        因为我们知道HashMap扩容时，容量会扩到原容量的2倍，
                        也就是放在链表中的Node的位置可能保持不变或位置变成 原位置+oldCap,在原位置基础上又加了一个数，位置变高了，
                        这里的高低位就是这个意思，低位指向的是保持原位置不变的节点，高位指向的是需要更新位置的节点
                    */
                    // Head指向的是链表的头节点，Tail指向的是链表的尾节点
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    // 指向当前遍历到的节点的下一个节点
                    Node<K,V> next;
                    //遍历链表
                    do {
                        //原索引
                        next = e.next;
                        /*
                            如果e.hash & oldCap == 0，注意这里是oldCap，而不是oldCap-1。
                            我们知道oldCap是2的次幂，也就是1、2、4、8、16...转化为二进制之后，
                            都是最高位为1，其它位为0。所以oldCap & e.hash 也是只有e.hash值在oldCap二进制不为0的位对应的位也不为0时，
                            才会得到一个不为0的结果。举个例子，我们知道10010 和00010 与1111的&运算结果都是 0010  ，
                            但是110010和010010与10000的运算结果是不一样的，所以HashMap就是利用这一点，
                            来判断当前在链表中的数据，在扩容时位置是保持不变还是位置移动oldCap。
                        */
                        // 如果结果为0，即位置保持不变
                        if ((e.hash & oldCap) == 0) {
                            // 如果是第一次遍历
                            if (loTail == null)
                                // 让loHead = e，设置头节点 
                                loHead = e;
                            else
                                // 否则,让loTail的next = e
                                loTail.next = e;
                            // 最后让loTail = e 
                            loTail = e;
                        }
                        /*
                            其实if 和else 中做的事情是一样的，本质上就是将不需要更新位置的节点加入到loHead为头节点的低位链表中，将需要更新位置的节点加入到hiHead为头结点的高位链表中。
                            我们看到有loHead和loTail两个Node,loHead为头节点，然后loTail是尾节点，在遍历的时候用来维护loHead，即每次循环，
                            更新loHead的next。我们来举个例子，比如原来的链表是A->B->C->D->E。
                            我们这里把->假设成next关系，这五个Node中，只有C的hash & oldCap != 0 ,
                            然后这个代码执行过程就是:
                            第一次循环： 先拿到A，把A赋给loHead,然后loTail也是A
                            第二次循环： 此时e的为B，而且loTail != null,也就是进入上面的else分支，把loTail.next =                     
                                        B，此时loTail中即A->B,同样反应在loHead中也是A->B,然后把loTail = B
                            第三次循环： 此时e = C，由于C不满足 (e.hash & oldCap) == 0,进入到了我们下面的else分支，其 
                                        实做的事情和当前分支的意思一样，只不过维护的是hiHead和hiTail。
                            第四次循环： 此时e的为D，loTail != null,进入上面的else分支，把loTail.next =                     
                                        D，此时loTail中即B->D,同样反应在loHead中也是A->B->D,然后把loTail = D
                        */
                        else {
                            if (hiTail == null)
                               hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                   } while ((e = next) != null);
                    // 遍历结束，即把table[j]中所有的Node处理完
                    // 如果loTail不为空，也保证了loHead不为空
                     if (loTail != null) {
                        // 此时把loTail的next置空，将低位链表构造完成
                        loTail.next = null;
                        // 把loHead放在newTab数组的第j个位置上，也就是这些节点保持在数组中的原位置不变
                        newTab[j] = loHead;
                     }
                     // 同理，只不过hiHead中节点放的位置是j+oldCap
                     if (hiTail != null) {
                        hiTail.next = null;
                        // hiHead链表中的节点都是需要更新位置的节点
                        newTab[j + oldCap] = hiHead;
                     }
                }
            }
        }
    }
    //返回新的表
    return newTab;
}
```

---

### 3.5 get()方法

get方法调用内部默认方法getNode获取值，查询到返回值，查询不到返回null

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

以下为getNode方法源码

```java
/*
* 按照传入的hash码和key值查询节点，并且返回节点
*/
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; //存储数组的临时变量
    Node<K,V> first, e; // first存储数组index的第一个Node节点
    int n; K k;
    //(tab = table) != null 当前table不能为空
    //(n = tab.length) > 0 table的长度必须大于0
    //(first = tab[(n - 1) & hash]) != null 查找index位置的第一个节点不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //first.hash == hash 如果第一个节点的hash与查找的hash值相同
        //(k = first.key) == key first的键和查找的键地址相同
        //(key != null && key.equals(k)) 键值不为空，并且first的键内容和key内容相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;//返回第一个值
        //没有找到目标
        //如果(e = first.next) != null first含有下一个节点并不为空，将其赋值给e
        if ((e = first.next) != null) {
            //如果第一个节点是红黑树
            if (first instanceof TreeNode)
                //调用getTreeNode方法
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //否则
            //遍历查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    //查不到 返回null
    return null;
}
```

---

### 3.6 remove()方法

remove()方法有两个

1.remove方法输入目标key值，存在就删除，并返回目标的值，不存在就返回null

2.remove方法输入目标key值，存在就删除，并返回true，不存在就返回false

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
//---
@Override
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}
```

remove方法调用了hashmap内的默认方法removeNode()

以下是removeNode()的源码

```java
/**
    移除某个节点，根据下面四个条件进行移除
    hash - key 的hash值 
    key - key
    matchValue - 如果为true，则仅在值相等时删除；如果是false，则值不管相不相等，只要key和hash值一致就移除该节点。
    movable - 如果为false，则在删除时不移动其他节点
    return - 返回被移除节点，未找到则返回null
*/
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab;//当前table数组的临时存储变量
    Node<K,V> p; //table[index]的第一个节点/遍历的前节点
    int n, index;//n:数组长度，index:数组的下标
    //(tab = table) != null && (n = tab.length) > 0 判断数组非空
    //(p = tab[index = (n - 1) & hash]) != null 查询要删除的节点位于数组中的下标，并将下标的第一位交给临时变量p
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //node被移除的节点
        //e指向第一个节点p的下一个节点
        Node<K,V> node = null, e;
        K k; V v;
        //如果第一个节点p就是目标节点，就将node指向p
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        //否则
        //如果p的下一个节点不为空，将其赋值给e
        else if ((e = p.next) != null) {
            //判断第一个节点p是否是数节点
            if (p instanceof TreeNode)
                //调用getTreeNode方法
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            //如果不是树节点
            else {
                //循环查找需要删除的节点
                do {
                    //如果查找到就将node指向e 并退出
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //找到目标节点了
        //matchValue为true，则仅在值相等时删除。如果是false，则值不管相不相等，只要key和hash值一致就移除该节点。
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            //如果是红黑树就调用树的删除方法
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            //如果node 是table[index]的第一个节点
            else if (node == p)
                //就跳过p
                tab[index] = node.next;
            else
                //否则p作为前节点，直接跳过node节点，指向下一个
                p.next = node.next;
            //记录操作次数
            ++modCount;
            //较小大小
            --size;
            afterNodeRemoval(node);
            //返回被删除节点
            return node;
        }
    }
    return null;
}
```

---

## 4.TreeNode源码

