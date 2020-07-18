---
title: ConcurrentHashMap 详解
date: 2020-05-17 15:43:05
categories: java
tags: [android, hashMap]
---

问题：
1. 使用 Hashmap 进行 put 操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。
为什么会发生死循环？应该如何避免？

```
void transfer(Entry[] newTable, boolean rehash) {
         int newCapacity = newTable.length;
         for (Entry<K,V> e : table) {
             while(null != e) {
                 Entry<K,V> next = e.next;
                 if (rehash) {
                     e.hash = null == e.key ? 0 : hash(e.key);
                 }
                 int i = indexFor(e.hash, newCapacity);
                 e.next = newTable[i];
                 newTable[i] = e;
                 e = next;
             }
         }
     }
```
在多线程环境下，reset 方法中的 trasnsfer 函数中的复制，可能会导致链表成环。
因为在 while 循环中，会通过头插法，反序当前链表的顺序，在执行过程中可能会被另外一个线程错误的赋值，从而导致死循环。


2. `HashTable` 利用`synchronized`关键字来证原子性，但是效率差。
效率为什么会会差
答：因为所有访问`HashTable`的线程都必须竞争同一把锁

3. JDK 8 中`ConcurrentHashMap` 是怎么保证多线程的效率的呢？
答：通过 CAS 原子锁 + node + sync 来保证。

4. JDK 7 `ConcurrentHashMap`实现远离
答：segment 分段锁，size（）这类全局都需要锁住的元素会先按顺序获取分段锁，如果获取失败，在重试两次后，会获取全局 synchronized 锁。
