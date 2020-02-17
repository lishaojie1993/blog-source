---
title: ConcurrentHashMap底层实现
date: 2019-03-19 22:54:49
categories: 
  - Java
  - 数据结构
tags:
  - 数据结构
  - 高并发
  - ConcurrentHashMap
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## Segment

Segment本身就相当于一个HashMap对象，同HashMap一样，Segment包含一个HashEntry数组，数组中的每一个HashEntry既是一个键值对，也是一个链表的头节点。像这样的Segment对象，在ConcurrentHashMap集合中有2的N次方个，共同保存在一个名为segments的数组当中。

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gaza7kcjxqj30kw0bsgn7.jpg)

<!-- More -->

可以说，ConcurrentHashMap是一个二级哈希表。在一个总的哈希表下面，有若干个子哈希表。这样的二级结构，和数据库的水平拆分有些相似。

ConcurrentHashMap优势就是采用了[锁分段]技术，每一个Segment就好比一个自治区，读写操作高度自治，Segment之间互不影响。

### 并发读写的情况

1. 不同Segment的并发写入——**可以并发执行**

   ![](https://tva1.sinaimg.cn/large/006tNbRwgy1gazahwt8gmj30hs083jrj.jpg)

2. 同一Segment的一写一读——**可以并发执行**

   ![](https://tva1.sinaimg.cn/large/006tNbRwgy1gazai5be4aj30jv07wgmh.jpg)

3. 同一Segment的并发写入——**会发生阻塞**

   ![](https://tva1.sinaimg.cn/large/006tNbRwgy1gazaicuv2mj30k1081q3s.jpg)

Segment的写入是需要上锁的，因此对同一Segment的并发写入会被阻塞。

由此可见，ConcurrentHashMap当中每个Segment各自持有一把锁。在保证线程安全的同时降低了锁的粒度，让并发操作效率更高。

## ConcurrentHashMap读写过程

### Get方法

1. 为输入的Key做Hash运算，得到hash值。
2. 通过hash值，定位到对应的Segment对象
3. 再次通过hash值，定位到Segment当中数组的具体位置。

### Put方法

1. 为输入的Key做Hash运算，得到hash值。
2. 通过hash值，定位到对应的Segment对象
3. 获取可重入锁
4. 再次通过hash值，定位到Segment当中数组的具体位置。
5. 插入或覆盖HashEntry对象。
6. 释放锁。

**从步骤中可以看出，ConcurrentHashMap在读写时都需要二次定位。首先定位到Segment，然后定位到Segment内的具体数组下标。**

## ConcurrentHashMap的Size方法

### 实现逻辑

ConcurrentHashMap的Size方法是一个嵌套循环，大体逻辑如下：

1. 遍历所有的Segment。
2. 把Segment的元素数量累加起来。
3. 把Segment的修改次数累加起来。
4. 判断所有Segment的总修改次数是否大于上一次的总修改次数。如果大于，说明统计过程中有修改，重新统计，尝试次数+1；如果不是。说明没有修改，统计结束。
5. 如果尝试次数超过阈值，则对每一个Segment加锁，再重新统计。
6. 再次判断所有Segment的总修改次数是否大于上一次的总修改次数。由于已经加锁，次数一定和上次相等。
7. 释放锁，统计结束。

### 官方代码

```java
public int size() {
    // Try a few times to get accurate count. On failure due to
   // continuous async changes in table, resort to locking.
   final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```

### 设计思想

这种思想和乐观锁悲观锁的思想如出一辙。

为了尽量不锁住所有Segment，首先乐观地假设Size过程中不会有修改。当尝试一定次数，才无奈转为悲观锁，锁住所有Segment保证强一致性。

## ConcurrentHashMap的扩容机制

当ConcurrentHashMap中元素的数量达到cap * loadFactor时，就需要进行扩容。扩容主要通过transfer()方法进行，当有线程进行put操作时，如果正在进行扩容，可以通过helpTransfer()方法加入扩容。也就是说，ConcurrentHashMap支持多线程扩容，多个线程处理不同的节点。

## JDK1.8下做了哪些优化

### 优势

JDK1.8 放弃了分段锁Segment而是用了Node，采用了 CAS + synchronized 来保证并发安全性，降低锁的粒度，提高性能，并使用CAS操作来确保Node的一些操作的原子性，取代了锁。

### 缺陷

ConcurrentHashMap的一些操作使用了synchronized锁，而不是ReentrantLock,虽然说JDK1.8的synchronized的性能进行了优化，但是我觉得还是使用ReentrantLock锁能更多的提高性能。

