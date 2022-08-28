# 源码分析之 LinkedHashMap

基于 Java 9

先学习一下 HashMap

HashMap 是无序的

## 特性

-  继承 HashMap 类，实现了 Map 接口。与 HashMap 不同之处是他通过双向链表**保证元素的迭代顺序，迭代顺序是插入顺序或者访问顺序**
- 如果某个 key 被重复插入，并不会改变这个 key 
- 插入的 key 和 vaule都可以为 null
- 非线程安全

## 基本结构

`LinkedHashMap` 继承`HashMap`类，实现了`Map`接口

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
```

### 属性

```java

    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * The iteration ordering method for this linked hash map: true
     * for access-order, false for insertion-order.
     * 
     * 为true时遍历集合会根据访问顺序，false时根据插入的顺序
     */
    final boolean accessOrder;
```

### 构造方法

一共有五个构造方法

#### 构造方法3

```java
public LinkedHashMap() {
    super();
    accessOrder = false;
}
```

无参的构造方法，accessOrder 为 false，迭代的顺序时插入元素的顺序

#### 构造方法4

```java
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}
```

支持传入一个 Map。putMapEntries 会遍历传入的 map，将元素添加到 LinkedHashMap 中

#### 构造方法5

```java
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

- initialCapacity：指定初始划容量。这是父类 HashMap  的属性，默认是16
- loadFactor：指定负载因子。这是父类 HashMap  的属性，默认是0.75
- accessOrder：指定排序的规则

## put 方法

LinkedHashMap 这个类中是没有 put 方法的，所以会调用父类 HashMap 的 put 方法。那 LinkedHashMap 怎么实现双向链表结构呢？看下源码

```java
// HashMap.java 
// 因为这一段代码是 HashMap 的 put 方法，与双向 

// 在 put方法中调用 putVal 方法，直接看下边的 putVal
public V put(K key, V value) {
    // hash(key) 是计算 key 的 hash 值
    return putVal(hash(key), key, value, false, true);
}

/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode. 主要关注一下这个参数
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 第一次 put 时会触发 resize，会对集合初始化，设置数组的长度，初始划容量及负载因子这些
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 的值是要添加的 key 在数组中的下标，如果这个位置没有值，就在这里新建一个 Node 放在这里
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else { // 如果这个位置有值，说明 hash 冲突，那么这个值可能就要插入链表或者红黑树了
        Node<K,V> e; K k;
        // 首先判断数组这个位置第一个元素的 key 跟我们要插入的是不是相等的，如果是就把这个节点取出来
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 在判断该节点是代表红黑树的节点，调用红黑树的插值方法
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else { // 如果都不是说明是链表中的节点
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // 插入到链表的最后
                    p.next = newNode(hash, key, value, null);
                    // 如果链表的长度大于等于8，就扩容为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果在链表中找到了相同的 key,直接跳出循环，e 就是链表中相同的那个节点  
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // e != null 说明要插入的key是已经存在的
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 如果参数 onlyIfAbsent 为 true ,表示只有key存在就不改变原来的值 
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // afterNodeAccess 方法体是空的，子类可以重写这个方法
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

