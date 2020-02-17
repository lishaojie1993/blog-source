---
title: 链表“逆序”算法
date: 2019-06-18 22:06:48
categories: 
  - Java
  - 算法
tags:
  - 算法
  - 链表逆序算法
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## 逆序本质

链表逆序的本质就是把每一个节点原本指向下一个节点的next指针倒转过来，指向它的前置节点。

## 实现步骤

1. 从链表头部开始，建立三个临时节点的引用，分别为p1，p2，p3。它们分别指向头节点、第二个节点和第三个节点。
2. 以p2节点为视角，把p2节点原本指向p3的next指针倒转，指向p1。
3. 三个临时节点的引用分别向后移动一格位置。
4. 重复第2步的工作，以p2节点为视角，把p2节点原本指向p3的next指针倒转，指向p1。
5. 重复第3步的工作，三个临时节点分别向后移动一格位置。
6. 继续这样迭代下去，直到p2是空为止。
7. 最后把head节点的next指向null，成为逆序链表的尾节点，并且把p1赋值给head，成为头节点。

<!-- More -->

## 代码实现

```java
private static Node head; //这里head是静态成员，其实也可以作为方法参数传入
public static void reverseLinkedList(){
  if(head==null || head.next==null){
    return;
  }
  //算法实现
  Node p1 = head;
  Node p2 = head.next;
  Node p3 = null;
  while(p2!=null){
    p3 = p2.next; //给p3赋值
    p2.next = p1; //p2指向p1
    p1 = p2; //给p1赋值
    p2 = p3; //给p2赋值
  }
  head.next = null; //head指向null
  head = p1; //给head赋值
}

//定义节点
private static class Node{
  int data;
  Node next;
  Node(int data){
    this.data = data;
  }
}
public static void main(String[] args){
  //初始化链表
  head = new Node(3); //给头节点赋值
  head.next = new Node(6); //头节点的next指向新节点6
  
  Node temp = head.next; //定义临时节点并赋值
  temp.next = new Node(1); //临时节点的next指向新节点1
  temp = temp.next; //临时节点指针后移
  temp.next = new Node(4);
  temp = temp.next;
  temp.next = new Node(9);
  
  //输出初始化的链表
  temp = head;
  while(temp!=null){
    System.out.println(temp.data);
    temp = temp.next;
  }
  
  //调用逆序链表的方法
  reverseLinkedList();
  
  //输出逆序后的链表
  temp = head;
  while(temp!=null){
    System.out.println(temp.data);
    temp = temp.next;
  }
}
```