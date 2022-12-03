# ArrayList详解


# 1.ArrayList<E>

---

## 1.1 简介

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{}
```

ArrayList是java集合类中比较常使用的集合类。该类继承了AbstractList<E>,实现类List<E>接口，底层基于数组实现容量大小动态变化。允许 null 的存在。同时还实现了 RandomAccess、Cloneable、Serializable 接口，所以ArrayList 是支持快速访问、复制、序列化的。

---

## 1.2 成员变量

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    //序列化ID
    private static final long serialVersionUID = 8683452581122892189L;
    //默认容量
    private static final int DEFAULT_CAPACITY = 10;
    //在有参构造函数中被构造
    private static final Object[] EMPTY_ELEMENTDATA = {};
    //在默认构造函数中被构造
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //维护的数据数组
    transient Object[] elementData; 
    //实际集合内元素数量
    private int size;
}
```

---

## 1.3 构造方法

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    //默认构造方法
    public ArrayList() {
        //赋值一个空数组
        //在add中重新修改数组大小
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    //指定容量的构造方法
    public ArrayList(int initialCapacity) {
        //判断容量是否大于0
        if (initialCapacity > 0) {
            //按照指定的容量创建一个数组，将其赋值给elementData
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //指定的容量为0，直接将成员变量赋值给elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //case <0 报错
            throw new IllegalArgumentException("Illegal Capacity: "+
                                              initialCapacity);
        }
    }
    //传入一个集合类
    public ArrayList(Collection<? extends E> c) {
        //将集合类转为Object数组
        Object[] a = c.toArray();
        //判空
        if ((size = a.length) != 0) {
            //如果类类型相等
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {//否则调用Arrays.copyOf拷贝数组
                    //底层调用System.arraycopy()
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            //指定的容量为0，直接将成员变量赋值给elementData
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
}
```

---

## 1.4 add(E e)

```java
    //ArrayList内的add()方法
    //将实际元素大小size+1传入ensureCapacityInternal
    //扩容后的数组在size++位置添加元素
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!
        elementData[size++] = e;
        return true;
    }
    //
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    //该方法判断传入elementData是否是默认的DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    //DEFAULTCAPACITY_EMPTY_ELEMENTDATA该数组代表调用了默认构造方法
    //之后判断默认的大小和传入的大小的大小，返回最大值
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    //接收到calculateCapacity传入的容量之后
    //操作次数++
    //如果传入的最小容量-当前数组中的大小 > 0 判定必须扩容
    //调用grow()函数 
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

---

## 1.5 grow(int minCapacity)

如果1.5倍扩容太小，则直接将所需的Capacity赋值给newCapacity

否则太大的话，需要做判断

1. 如果扩容之后的值超过了Integer.MAX_VALUE - 8，就调用hugeCapacity()

```java
    //扩容函数
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //                0.5倍       +    1倍    =     1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果扩容的容量还是比minCapacity小
        //直接newCapacity = minCapacity;
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果扩容的容量比MAX_ARRAY_SIZE大
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    //数组最大容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

---

## 1.6 add的其他方法

```java
   //指定index插入元素
   public void add(int index, E element) {
        rangeCheckForAdd(index);//检查范围是否合法
       //接下来和add()无差别
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
   //添加集合内全部元素
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
       //从数组a的下标0开始，复制到elementData，从下标size位置，复制数量为numNew
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
   //添加集合内全部元素，到指定下标
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);//检查index合法性
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //将[index,size]范围的元素 后移numNew位
        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
        //拷贝
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
```

---

## 1.7 rangeCheck(int index)

检查范围

```java
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

检查范围for add()

```java
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

---

## 1.8 remove()

```java
    //删除指定下标的元素
    public E remove(int index) {
        rangeCheck(index);//检查下标合法性
        modCount++;//操作数++
        E oldValue = elementData(index);//获取被删除元素
        int numMoved = size - index - 1;//判断被删除元素是否是最后一个
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
        //返回删除值
        return oldValue;
    }
    //按照输入参数删除元素
    public boolean remove(Object o) {
        //case1 删除空值
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        //case2 删除非空值
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    //fastremove()和remove()基本相同
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

---

## 1.9 clear()

```java
   //清空内部所有元素
     public void clear() {
        modCount++;
        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

---

## 1.10 get()

```java
    public E get(int index) {
        rangeCheck(index);//检查范围
        return elementData(index);
    }
    //返回对应下表的内容
    E elementData(int index) {
        return (E) elementData[index];
    }
```

---

## 1.11 set()

```java
    public E set(int index, E element) {
        rangeCheck(index);//检查范围
        //获取被修改的值
        E oldValue = elementData(index);
        elementData[index] = element;//修改对应下标的内容
        return oldValue;//返回旧值
    }
```

---

## 1.12 迭代器

在用 for 遍历集合的时候是不可以对集合进行 remove操作的，因为 remove 操作会改变集合的大小。从而容易造成结果不准确甚至数组下标越界，更严重者还会抛出 ConcurrentModificationException。

所以ArrayList内部定义了一个实现Iterator的接口，来实现remove()

```java
 /**
     * An optimized version of AbstractList.Itr
     */
    private class Itr implements Iterator<E> {
        int cursor;       // 下一个需要访问的元素下标
        int lastRet = -1; // 上一个访问过的元素
        int expectedModCount = modCount;//操作次数

        Itr() {}
        //只需要判断下一个下标是不是size即可
        public boolean hasNext() {
            return cursor != size;
        }
        //返回下一个
        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();//检测初期操作次数
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
        //删除
        public void remove() {
             //上一个lastRet小于0
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();//检查操作次数

            try {
                ArrayList.this.remove(lastRet);//删除上一次访问的数值
                cursor = lastRet;
                lastRet = -1;//重置为-1
                expectedModCount = modCount;//操作次数赋值
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
         @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
 }
```

