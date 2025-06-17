---
layout: post
title: JDK Collection.md
categories: [Java]
description: Java
keywords: Java
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Collection

## 概述

## 源码解析

### 方法定义

```java
// @since 2
public interface Collection<E> extends Iterable<E> {
    // @since 8 移除集合中所有符合断言的元素
	default boolean removeIf(Predicate<? super E> filter) {}
    // @since 8
    default Stream<E> stream() {}
    // @since 8
    default Stream<E> parallelStream() {}
}
```



## 实践应用



# SequencedCollection

## 概述

> @since 21



JDK 21引入了一种新的集合类型：**Sequenced Collections（序列化集合，也叫有序集合）**，这是一种具有确定出现顺序（encounter order）的集合，无论遍历这样的集合多少次，元素的出现顺序始终是固定的。序列化集合提供了处理集合的第一个和最后一个元素以及反向视图（与原始集合相反的顺序）的简单方法。Sequenced Collections 包括以下三个接口：

- `SequencedCollection`

- `SequencedSet`

- `SequencedMap`

  

## 源码解析

### 方法定义

```java
public interface SequencedCollection<E> extends Collection<E> {
    // @since 21
	SequencedCollection<E> reversed();
    // @since 21
    default void addFirst(E e) {}
    // @since 21
    default void addLast(E e) {}
    // @since 21
    default E getFirst() {}
    // @since 21
    default E getLast() {}
    // @since 21
    default E removeFirst() {}
    // @since 21
    default E removeLast() {}
}
```



# SequencedSet

## 概述

## 源码解析

### 方法定义

```java
public interface SequencedSet<E> extends SequencedCollection<E>, Set<E> {
	// @since 21
    SequencedSet<E> reversed();
}
```



# List

## 概述

`List`：存储的元素是有序可重复的。

List

- ArrayList 基于动态数组实现，支持随机访问。
- Vector 和 ArrayList 类似，但它是线程安全的。
- LinkedList 基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

Queue

- LinkedList 可以用它来实现双向队列。
- PriorityQueue 基于堆结构实现，可以用它来实现优先队列。



## 源码解析

### 方法定义

```java
public interface List<E> extends SequencedCollection<E> {
    // @since 8 替换集合中所有的元素为转换后的新值，新值类型和当前元素类型一致
	default void replaceAll(UnaryOperator<E> operator) {}
    // @since 8 集合中所有元素根据指定比较器设置的规则进行排序
    default void sort(Comparator<? super E> c) {}
    // @since 9 使用静态工厂方法来创建不可变集合
    static <E> List<E> of() {}
    // @since 9 根据已有集合来创建不可变集合
    static <E> List<E> copyOf(Collection<? extends E> coll) {}
}
```



### 方法说明

#### of

> @Since Java 9



使用`of()`创建的集合为不可变集合，进行添加、删除、替换、排序等操作时会报`java.lang.UnsupportedOperationException`异常。

```java
static <E> List<E> of(E e1, E e2) {
        return new ImmutableCollections.List12<>(e1, e2);
}
static <E> List<E> of(E e1, E e2, E e3) {
    return ImmutableCollections.listFromTrustedArray(e1, e2, e3);
}
// ... 其他重载of方法
```



#### copyOf

> @Since Java 10



使用`copyOf()`创建的集合为不可变集合，进行添加、删除、替换、排序等操作时会报`java.lang.UnsupportedOperationException`异常。

```java
static <E> List<E> copyOf(Collection<? extends E> coll) {
    return ImmutableCollections.listCopy(coll);
}
```



# Set

## 概述

`Set`：存储的元素是无序不可重复的。

- TreeSet 基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
- HashSet 基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
- LinkedHashSet 具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。



## 源码解析

### 方法

| 方法名称 | 方法描述                         |
| -------- | -------------------------------- |
| of       | 使用静态工厂方法来创建不可变集合 |
| copyOf   | 根据已有集合来创建不可变集合     |



#### of

> @Since Java 9



使用`of()`创建的映射为不可变映射，进行添加、删除、替换、排序等操作时会报`java.lang.UnsupportedOperationException`异常。

```java
static <E> Set<E> of(E e1) {
    return new ImmutableCollections.Set12<>(e1);
}

static <E> Set<E> of(E e1, E e2) {
    return new ImmutableCollections.Set12<>(e1, e2);
}

// ... 其他重载of方法
```



#### copyOf

> @Since Java 9



使用`copyOf()`创建的映射为不可变映射，进行添加、删除、替换、排序等操作时会报`java.lang.UnsupportedOperationException`异常。

```java
static <E> Set<E> copyOf(Collection<? extends E> coll) {
    if (coll instanceof ImmutableCollections.AbstractImmutableSet) {
        return (Set<E>)coll;
    } else {
        return (Set<E>)Set.of(new HashSet<>(coll).toArray());
    }
}
```



# ArrayList

*ArrayList*实现了*List*接口，是顺序容器，即元素存放的数据与放进去的顺序相同，允许放入`null`元素，底层通过**数组实现**。除该类未实现同步外，其余跟*Vector*大致相同。每个*ArrayList*都有一个容量(capacity)，表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。

ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。



> java.util.ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空集合对象共享的空数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList大小，即包含的元素数量
     */
    private int size;
  	// ...
}
```



ArrayList底层是一个Object数组。



### 最佳实践

#### 删除元素

- 跳过fast-fail检测操作，使用普通for循环；
- Collection的removeIf方法，通过iterator操作；
- Stream的filter方式；
- 使用fail-safe的集合类。



### 源码解析

#### 构造方法

> java.util.ArrayList

```java
    /**
     * 指定初始容量参数的构造函数
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // 初始容量大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
         	  // 初始容量等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // 初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 集合参数为空集合时，用默认空数组替换
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```



#### 属性

##### elementData

> java.util.ArrayList

```java
transient Object[] elementData;
```



该属性存储集合元素，空集合时该值为空数组，添加第一个元素时进行扩容

`transient`关键字修饰该属性是为了序列化时不将其中的空对象进行序列化，`ArrayList`通过`writeObject`和`readObject`重新实现了序列化的逻辑来只序列化集合中实际存在的元素。



#### 添加元素

> java.util.ArrayList

```java
    /*
     * 将指定的元素追加到此列表的末尾。
     */
		public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 增加modCount!!
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // 集合数组为空时，比较默认容量和容量参数，返回较大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
      	// 直接返回容量参数
        return minCapacity;
    }
```



##### **判断是否扩容**

> java.util.ArrayList

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            // 调用扩容方法
            grow(minCapacity);
    }
```



- 没有指定默认容量时，添加第一个元素时，minCapacity为10，elementData.length为0，此时判断为true，进行扩容；

- 添加元素直到第11个元素时，minCapacity为11，elementData.length为10，此时再次进行扩容；
- 以此类推······



##### **扩容**

每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过ensureCapacity(int minCapacity)方法来实现。在实际添加大量元素前，我也可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的1.5倍。这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用ensureCapacity方法来手动增加ArrayList实例的容量。



> java.util.ArrayList

```java
    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

	  /**
     * ArrayList扩容的核心方法
     * 增加容量以确保它至少可以容纳最小容量参数指定的元素数量。
     */
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 扩容后的容量是旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 检查新容量是否大于最小需要容量，若小于最小需要容量，就把最小需要容量当作数组新容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果新容量大于 MAX_ARRAY_SIZE，则比较 minCapacity 和 MAX_ARRAY_SIZE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        // 如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即 `Integer.MAX_VALUE - 8`
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



- 当add第1个元素时，oldCapacity 为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断不会成立，即newCapacity 不比 MAX_ARRAY_SIZE大，则不会进入 `hugeCapacity` 方法。数组容量为10，add方法中 return true,size增为1。
- 当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true,size增为11。
- 以此类推······



#### 删除元素

##### 删除指定元素

1. for循环匹配传入的值；
2. 调用fastRemove(index)删除数据，将index + 1及之后的元素向前移动一位，覆盖被删除值；
3. 清空最后一个元素。



##### 删除指定位置元素

1. 范围检测是否越界；
2. 取出旧数据；
3. 调用fastRemove(index)删除数据，将index + 1及之后的元素向前移动一位，覆盖被删除值；
4. 清空最后一个元素。



##### 缩容

1. 当容量为0时，重置为空数组；
2. 容量小于数组缓冲区容量，创建一个新的数组拷贝过去。



#### 手动扩容

> java.util.ArrayList

```java
    /**
     * 增加此list对象的容量，以确保它可以容纳由minimum capacity参数指定的元素数
     *
     * @param   minCapacity   the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        // 数组不为默认空数组时，返回0，否则返回默认容量10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;
        if (minCapacity > minExpand) {
            // 执行判断是否扩容，根据需要对集合进行扩容操作
            ensureExplicitCapacity(minCapacity);
        }
    }
```



在添加大量元素前，调用该方法来减少重新扩容的次数，提高代码执行效率。



#### 转换为数组

> java.util.ArrayList

```java
    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 返回的数组的运行时类型是指定数组的运行时类型。
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 创建一个运行时类型的新数组
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```



#### 序列化和反序列化

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

序列化该集合，将集合中数据写入流。



```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

反序列化该集合，从流中按顺序获取数据。



### 扩展

#### RandomAccess 接口

> java.util.RandomAccess

```java
public interface RandomAccess {
}
```



`RandomAccess` 接口中什么都没有定义。所以该接口仅用于标识实现这个接口的类具有随机访问功能。

在 `binarySearch（)` 方法中，它要判断传入的 list 是否 `RamdomAccess` 的实例，如果是，调用`indexedBinarySearch()`方法，如果不是，那么调用`iteratorBinarySearch()`方法。



> java.util.Collections

```java
public static <T>
int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}

private static <T>
int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
    int low = 0;
    int high = list.size()-1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        Comparable<? super T> midVal = list.get(mid);
        int cmp = midVal.compareTo(key);

        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found
}

private static <T>
int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key)
{
    int low = 0;
    int high = list.size()-1;
    ListIterator<? extends Comparable<? super T>> i = list.listIterator();

    while (low <= high) {
        int mid = (low + high) >>> 1;
        Comparable<? super T> midVal = get(i, mid);
        int cmp = midVal.compareTo(key);

        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found
}
```



ArrayList 实现了 RandomAccess 接口， 而 LinkedList 没有实现。ArrayList底层是数组，而LinkedList底层是链表。数组天然支持随机访问，时间复杂度为 O(1)，所以称为快速随机访问。链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，所以不支持快速随机访问。ArrayList实现了RandomAccess接口，就表明了他具有快速随机访问功能。RandomAccess接口只是标识，并不是说ArrayList实现RandomAccess` 接口才具有快速随机访问功能的！



# LinkedList

> java.util.LinkedList

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
}
```



LinkedList 底层是一个双向链表。



### 源码解析

#### 获取元素

> java.util.LinkedList

```java
    public E get(int index) {
      	// 首先判断下标是否为负数或越界
        checkElementIndex(index);
        return node(index).item;
    }

    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

    Node<E> node(int index) {
      	// 判断下标属于前半部分还是后半部分，决定是从前往后遍历还是从后往前遍历
        if (index < (size >> 1)) {
          	// 从前往后遍历
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
          	// 从后往前遍历
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }

```



### ArrayList和LinkedList比较

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。）
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** ArrayList 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。



# Vector

`ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；`Vector` 是 `List` 的古老实现类，底层使用` Object[ ]` 存储，线程安全的。



# HashSet

> java.util.HashSet

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;

    // 与支持 Map 中的对象关联的虚拟值
    private static final Object PRESENT = new Object();
}
```



`HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的，可以存储 null 值。



### 源码解析

#### 添加元素

> java.util.HashSet

```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```



HashSet添加元素时即调用了HashMap的put方法，此时如果map的key位置为空，则返回true；反之则返回旧值表示添加失败。



# LinkeHashSet



`LinkedHashSet` 是 `HashSet` 的子类，其内部是通过 `LinkedHashMap` 来实现，能够按照添加的顺序遍历。



# TreeSet



`TreeSet` 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。