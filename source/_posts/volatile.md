---
title: volatile关键字解析
date: 2019-03-15 22:56:30
categories: Java
tags: 
  - 高并发
  - volatile
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## volatile的两大特性

1. 保证变量在线程之间的可见性。可见性的保证是基于CPU的内存屏障指令，被JSR-133抽象为happens-before原则。（先行发生原则）
2. 阻止编译时和运行时的指令重排。编译时JVM编译器遵循内存屏障的约束，运行时依靠CPU屏障指令来阻止重排。

## Java内存模型JMM

Java内存模型(即Java Memory Model，简称JMM)本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。<!-- More -->

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，用于存储线程私有的数据，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷贝，前面说过，工作内存是每个线程的私有数据区域，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成。

### 主内存（Main Memory）

主内存可以简单理解为计算机当中的内存，但又不完全等同。主内存被所有的线程所共享，对于一个共享变量（比如静态变量，或是堆内存中的实例）来说，主内存当中存储了它的“本尊”。

### 工作内存（Working Memory）

工作内存可以简单理解为计算机当中的CPU高速缓存，但又不完全等同。每一个线程拥有自己的工作内存，对于一个共享变量来说，工作内存当中存储了它的“副本”。

### 为什么所有的线程不直接操作主内存？

直接操作主内存太慢了，工作内存类似于高速缓存，访问速度更快。

## volatile的出现

多线程的情况下，由于工作内存所更新的变量不会立即同步到主内存，为了防止产生错误的结果，可以使用同步锁来保证线程安全，但是对程序的性能影响太大，所以产生了volatile。

### volatile的可见性

当一个线程修改了变量的值，新的值会立刻同步到主内存当中。而其他线程读取这个变量的时候，也会从主内存中拉取最新的变量值。为什么volatile关键字可以有这样的特性？这得益于java语言的**先行发生原则**（happens-before）。

#### happens-before

在计算机科学中，先行发生原则是两个事件的结果之间的关系，如果一个事件发生在另一个事件之前，结果必须反映，即使这些事件实际上是乱序执行的（通常是优化程序流程）。

这里所谓的事件，实际上就是各种指令操作，比如读操作、写操作、初始化操作、锁操作等等。先行发生原则作用于很多场景下，包括同步锁、线程启动、线程终止、volatile。

#### volatile的适用范围

1. **运行结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。**

   ```java
   public class VolatileTest{
     public volatile static int count = 0;
     public static void main(String[] args){
       //开启10个线程
       for(int i=0;i<10;i++){
         new Thread(
         	new Runnable(){
             public void run(){
               try{
                 Thread.sleep(1);
               }catch(InterruptedException e){
                 e.printStackTrace();
               }
               //每个线程当中让count值自增100次
               for(int j=0;j<100;j++){
                 count++;
               }
             }
           }
         ).start();
       }
       try{
         Thread.sleep(2000);
       }catch(InterruptedException e){
         e.printStackTrace();
       }
       System.out.print("count="+count);
     }
   }
   ```

   例子中的实际值要小于1000.

2. **变量不需要与其他的状态变量共同参与不变约束。**

   ```java
   volatile static int start = 3;
   volatile static int end = 6;
   ```

   线程A执行如下代码：

   ```java
   while (start < end){
    //do something
   }
   ```

   线程B执行如下代码：

   ```java
   start+=3;
   end+=3;
   ```

   这种情况下，一旦在线程A的循环中执行了线程B，start有可能先更新成6，造成了一瞬间 start == end，从而跳出while循环的可能性。



#### 什么是指令重排？

指令重排是指JVM在编译Java代码的时候，或者CPU在执行JVM字节码的时候，对现有的指令顺序进行重新排序。

#### 指令重排的目的

为了在不改变程序执行结果的前提下，优化程序的运行效率。需要注意的是，这里所说的不改变执行结果，指的是不改变单线程下的程序执行结果。

### volatile的内存屏障

#### 什么是内存屏障？

内存屏障也称为内存栅栏或栅栏指令，是一种屏障指令，它使CPU或编译器对屏障指令之前和之后发出的内存操作执行一个排序约束。 这通常意味着在屏障之前发布的操作被保证在屏障之后发布的操作之前执行。

#### 内存屏障的 4 种类型

**LoadLoad屏障**：

抽象场景：Load1; LoadLoad; Load2

Load1 和 Load2 代表两条读取指令。在Load2要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

**StoreStore屏障：**

抽象场景：Store1; StoreStore; Store2

Store1 和 Store2代表两条写入指令。在Store2写入执行前，保证Store1的写入操作对其它处理器可见

**LoadStore屏障：**

抽象场景：Load1; LoadStore; Store2

在Store2被写入前，保证Load1要读取的数据被读取完毕。

**StoreLoad屏障：**

抽象场景：Store1; StoreLoad; Load2

在Load2读取操作执行前，保证Store1的写入对所有处理器可见。StoreLoad屏障的开销是四种屏障中最大的。

#### volatile如何防止指令重排

在一个变量被volatile修饰后，JVM会为我们做两件事：

1. 在每个volatile写操作前插入**StoreStore**屏障，在写操作后插入**StoreLoad**屏障。
2. 在每个volatile读操作前插入**LoadLoad**屏障，在读操作后插入**LoadStore**屏障。

从而成功阻止了指令重排序。