# 源码分析之 LinkedHashMap

以下源代码基于 Java 8，我看了一下 Java 9，Java 11应该都是这样的

因为 LinkedHashMap 是基于 HashMap 实现的，所以要先了解一下 HashMap 的基本属性，比如 HashMap 是由数组 + 链表或红黑树实现的；HashMap 是无序的

## 特性

-  继承 HashMap 类，实现了 Map 接口。与 HashMap 不同之处是他通过双向链表**保证元素的迭代顺序，迭代顺序是插入顺序或者访问顺序**
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
     * 双向链表的头节点
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     * 双向链表的尾节点
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
    // (n - 1) & hash 的值是要添加的 key 在数组中的下标，如果这个位置没有值，就在这里新建一个 Node 放在这里。LinkedHashMap 重写了 newNode 方法
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
            // afterNodeAccess 方法体是空的，其实就是给 LinkedHashMap 准备的，LinkedHashMap 重写了这个方法
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 如果插入新值后，size 超过阈值会触发扩容
    if (++size > threshold)
        resize();
    // 这个也是为 LinkedHashMap 准备的，方法体是空的，LinkedHashMap 重写了这个方法
    afterNodeInsertion(evict);
    return null;
}
```

### newNode 和 linkNodeLast 方法

```java
// LinkedHashMap.java

	// LinkedHashMap 重写了父类 HasMap 的 newNode 方法
	Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    // link at the end of list
	// 这个方法是把要插入的节点添加到双向链表的末尾
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }

```

### afterNodeInsertion  方法

```java
// LinkedHashMap.java

	// 执行 put 方法的最后会执行这个方法，目的是移除最老的节点，但是 LinkedHashMap 并不会移除，因为 removeEldestEntry 这个，方法一定返回是 false。 LinkedHashMap 的子类可以重写 removeEldestEntry 方法，就可以在控制在某个时机移除最老的节点。
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }

```

### afterNodeAccess 方法

```java
	// 如果要插入的 key 已经存在会调用这个方法，这个参数 e 是已经存在的那个节点
	// 这个方法主要作用是 如果要插入的节点已经存在了，那么就把原来已经存在的节点移动到双向链表的末尾
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        // accessOrder 为 true，即按照访问顺序
        // tail != e 即 e 不是尾节点
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            // p 是当前节点，是与要插入的 key 相同的已经存在的节点
            // b 是 p 的前驱节点
            // a 是 p 的后继节点
            // 三个节点对应关系为 b <-> p <-> a
            
            // 断开 p 与后一个节点的关联
            p.after = null;
            // p 的前驱节点为 null，说明 p 是双向链表中第一个节点，那么把 a 设为头节点
            if (b == null)
                head = a;
            else
                // 否则将 a 设为 b 的后继节点，也就是去掉了中间的 p 节点
                b.after = a;
            // 将 a 的前驱节点设为 b，即去掉了中间的 p 节点
            if (a != null)
                a.before = b;
            else
                // 此时将 b 设为最后一个节点
                last = b;
            
            // 下面就是将 p 放到双向链表的最后
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            // 将 p 设为双链表的尾节点
            tail = p;
            ++modCount;
        }
    }
```



综上可以看出，LinkedHashMap 插入会有两种情况

- 如果插入的 key 不存在，那么就会直接把插入的节点添加到双向链表的末尾，调用的是 LinkedHashMap 的 newNode() 方法
- 如果插入的 key 已经存在了，就会把原来已存在的接口转换到双向链表的末尾，调用的是 LinkedHashMap 的 afterNodeAccess() 方法

另外，如果想在插入的过程中删除最老的那个节点，比如想固定 size，当 size 到某个值的时候就删除最老的节点，可以重写afterNodeAccess() 方法

## get 方法

LinkdeHashMap 重写了 get() 方法，因为 LinkdeHashMap 支持根据访问顺序排列，就是当访问了一个元素，就把这个元素放到双向队列的末尾，这样就可以保证队列后边是最近才访问的，也就实现了 LRU(Least Recently Used) 最近最少使用算法

```java
//LinkdeHashMap.java
 	public V get(Object key) {
        Node<K,V> e;
        // getNode() 方法是父类 HashMap 的方法
        if ((e = getNode(hash(key), key)) == null)
            return null;
        // 如果是根据访问顺序排序，就把这个节点放到双向队列的末尾。在 put() 方法中当插入重复key时也会调用 afterNodeAccess()方法，上边有分析
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

### getNode 方法

```java
// HashMap.java

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 检查是不是数组中第一个节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                // 如果是红黑树，按红黑树的方式或者节点
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                
                // 如果是链表结构，就遍历链表获取节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```