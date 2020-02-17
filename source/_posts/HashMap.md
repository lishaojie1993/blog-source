---
title: HashMap底层原理
date: 2019-03-18 21:54:59
categories: 
  - Java
  - 数据结构
tags: 
  - 数据结构
  - HashMap
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## HashMap的底层原理

HashMap是一个用于存储key-value键值对的集合，每一个键值对也叫做Entry。数组中的每一个Entry元素，又是一个链表的头节点。这些键值对（Entry）分散存储在一个数组当中，这个数组就是HashMap的主干。

Hashmap不是线程安全的。在高并发环境下做插入操作，有可能出现环形链表。HashMap数组每一个元素的初始值都是Null，最常使用的两个方法是Put和Get。

### Put方法的原理

比如调用 hashMap.put("apple", 0) ，插入一个Key为“apple"的元素。这时候我们需要利用一个哈希函数：**index = Hash（“apple”）**来确定Entry的插入位置（index），但是，因为HashMap的长度是有限的，当插入的Entry越来越多时，再完美的Hash函数也难免会出现index冲突的情况。<!-- More -->

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gayr9a1hu4j30ji06fq3g.jpg)

这时我们可以通过**链表**来解决。需要注意的是，新来的Entry节点插入链表时，使用的是“头插法”，因为HashMap的发明者认为：**后插入的Entry被查找的可能性更大**。

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gayr9i7blpj30ju06wjrw.jpg)

### Get方法的原理

首先会把需要查询的key做一次Hash映射，得到对应的下标，由于刚才所说的Hash冲突，同一个位置有可能匹配到多个Entry，这时候就需要顺着对应链表的头节点，一个一个向下来查找。

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gayreaylimj30jf07tt9n.jpg)

## HashMap的特点

### HashMap的默认初始长度

HashMap的默认初始长度是16，并且每次自动扩展或是手动初始化时，长度必须是2的幂。

### 为什么这样设置？

为了使Hash算法得到的index尽可能均匀分布。例如从Key映射到HashMap数组的对应位置时，会用到一个Hash函数：**index = HashCode（Key） & （Length - 1）**，为了实现高效的Hash算法，采用位运算（与运算）

## 高并发下的HashMap

### Resize

HashMap的容量是有限的，当经过多次元素插入，使得HashMap达到一定饱和度时，key映射位置发生冲突的几率会逐渐提高。这时候，HashMap需要扩展它的长度，也就是进行**Resize**。

### 影响Resize的两个因素

1. Capacity

   HashMap的当前长度。

2. LoadFactor

   HashMap负载因子，默认值为0.75f。

衡量HashMap是否需要进行Resize的条件如下：

**HashMap.Size  >= Capacity \* LoadFactor**

### Resize做了什么？

1. 扩容

   创建一个新的Entry空数组，长度是原数组的2倍。

2. Rehash

   遍历原Entry数组，把所有的Entry重新Hash到新数组。

**为什么要重新Hash呢？**

因为长度扩大以后，Hash的规则也随之改变。（16=1111；32=11111）

**ReHash的Java代码如下：**

```java
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

### 为什么HashMap会出现死锁

在高并发情况下，如果HashMap正好达到临界点准备扩容，此时正好有两个线程在同时访问，在Rehash时链表可能会产生环形，当调用Get查找一个不存在的key，而这个key的Hash结果恰好位于环形链表处，这时程序将会进入死循环。

## JDK1.8下的HashMap优化

### 引入红黑树

我们知道，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦拉链过长则会严重影响HashMap的性能，所以JDK1.8版本引入了红黑树：当链表长度太长（默认超过8）时，链表就转为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能。

### Resize扩容优化

我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。