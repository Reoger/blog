---
title: LinkedHashMap实现LRUCache
date: 2018-11-12 23:14:55
categories: sourceCode
tags: [java,HashMap,LRUCache]
---

# LinkedHashMap实现LRUCache
---
## LinkedHashMap实现原理
简单介绍一下LinkedHasMap的实现原理，针对JDK 8.0，在不同的版本上其实现可能有所区别。
 
## 原理概括
``LinkedHasMap``就是基于``HashMap``，通过维护一个双向链表，达到在使用``HashMap``存储的情况下，记录其顺序。
``linkedHashMap``结构图的示意图如图所示：
![linkedHashMap结构图](https://images2015.cnblogs.com/blog/249993/201612/249993-20161215143120620-1544337380.png)

水平所限，本篇不会把所有的实现都呈现出来，只对其其中的几个关键性的方法函数进行解析。

## 构造方法
其构造方法主要有三个，只介绍其中的一个。

```
   public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

这里的 ``initialCapacity``表示初始的长度，``loadFactor``表示加载因子,``accessOrder``表示访问顺序，当其值为``true``时，表示当前的``LinkedHashMap``的顺序由访问数据时决定，即数据访问之后就会将这个数据放在``LinkedHashMap``数据项的前面来，而``accessOrder``为false时，则表示数据的访问数据由插入时就决定好了。

## put方法
往``LinkedHashMap``里面添加数据的方法就是通过``put``方法实现，在jdk 8.0中``LinkedHashMap``并没有自己实现``put``方法，而是由``HashMap``一同实现了。下面是具体的实现：
```
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
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

代码略长，长话短说，前面的大部分都是``hashMap``的实现，到了最后``afterNodeInsertion``，这个方法就是留给``LinkedHashMap``去实现其调换顺序的。我们直接看``LinkedHashMap``中这个方法的实现：
```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```
这个方法主要实现在某些情况下，需要将老的数据移除掉，移除就是通过``removeNode``实现的，我们不往下继续看，只关注其条件。我们观察到 需要移除老数据的添加有三个.
1. evict的boolean为true
2. head ！= null
3. removeEldestEntry(first) 返回true
其中，``evict``在调用时就是传入的值true,而``head``在``LinkedHashMap``中初始化并添加数据后，就不会为null了，所以这里需要直接移除老数据的关键条件就是``removeEldestEntry``这个方法了，而这个方法在``LinkedHashMap``的实现中默认是范围false的，即默认不用移除掉老的数据。那么当我们需要``LinkedHashMap``存储的数据达到一定量的时候，移除掉老数据就需要重写``removeEldestEntry``这个方法了。

## get
get的实现在``LinkedHashMap``重写了，实现也很简单，首先判断有没有这个key，如果有，在判断当前的``accessOrder``是不是为true，如果为true，则需要将顺序按照访问顺序调整一下，然后将数据返回回去。
```
 public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

### 利用``HashMap``实现LRUCache

```
class LRUCache {
    
    class Node {
        int key;
        int value;
        Node pre;
        Node next;
    }
    
    private int capacity;
    
    private Node head, tail;
    
    private Map<Integer, Node> map = null;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>(capacity);
        head = new Node();
        tail = new Node();
        head.pre = null;
        head.next = tail;
        tail.pre = head;
        tail.next = null;
    }
    
    public int get(int key) {
        Node node = map.get(key);
        if (node == null) {
            return -1;
        }
        moveToHead(node);
        return node.value;
    }
    
    private void moveToHead(Node node) {
        removeNode(node);
        addNode(node);
    }
    
    private void removeNode(Node node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
    }
    
    private void addNode(Node node) {
        node.pre = head;
        node.next = head.next;
        head.next.pre = node;
        head.next = node;
    }
    
    public void put(int key, int value) {
        Node node = map.get(key);
        if (node == null) {
            Node newNode = new Node();
            newNode.key = key;
            newNode.value = value;
            map.put(key, newNode);
            addNode(newNode);
            if (map.size() > capacity) {
                Node tail = popTail();
                map.remove(tail.key);
            }
        } else {
            node.value = value;
            moveToHead(node);
        }
    }
    
    private Node popTail() {
        Node node = tail.pre;
        removeNode(node);
        return node;
    }
    
}
```