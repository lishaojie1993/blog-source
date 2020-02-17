---
title: ZooKeeper相关
date: 2018-06-03 12:42:46
categories: Java
tags:
  - 分布式
  - zookeeper
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## zookeeper特点
### zookeeper是什么

zookeeper 是一个开源的分布式协调服务，是分布式数据一致性的解决方案。

zookeeper 本质上是一个分布式的小文件存储系统，提供基于类似于文件系统的目录树方式的数据存储，并且可以对树中的节点进行有效管理，从而用来维护和监控你存储的数据的状态变化，通过监控这些数据状态的变化，从而可以达到基于数据的集群管理。

### zookeeper能做什么

简单来说，zookeeper 主要做两件事，数据管理和监听服务。

分布式应用程序可以基于 zookeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、leader 选举、分布式锁和分布式队列等功能。

zookeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心。 服务生产者将自己提供的服务注册到 zookeeper 中心，服务的消费者在进行服务调用的时候先到 zookeeper 中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容与数据。如下图所示，在 Dubbo 架构中 zookeeper 就担任了注册中心这一角色。为了保证高可用，最好是以集群形态来部署 zookeeper，只要半数以上节点存活，zookeeper  就能正常服务。

<!-- more -->


zookeeper 主要提供下面几个功能：

1. 集群管理：容错、负载均衡。
2. 配置文件的集中管理。
3. 集群的入口。

### Session会话

Session 指的是 zookeeper  服务器与客户端会话。在 zookeeper 中，一个客户端连接是指客户端和服务器之间的一个 TCP 长连接。客户端启动的时候，首先会与服务器建立一个 TCP 连接，从第一次连接建立开始，客户端会话的生命周期也开始了。通过这个连接，客户端能够通过**心跳检测**与服务器保持有效的会话，也能够向 zookeeper 服务器发送请求并接受响应，同时还能够通过该连接接收来自服务器的**Watch事件通知**。 

Session 的 sessionTimeout 值用来设置一个客户端会话的超时时间。当由于服务器压力太大、网络故障或是客户端主动断开连接等各种原因导致客户端连接断开时，只要在 sessionTimeout 规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话仍然有效。

在为客户端创建会话之前，服务端首先会为每个客户端都分配一个 sessionID。由于 sessionID 是 zookeeper 会话的一个重要标识，许多与会话相关的运行机制都是基于这个 sessionID 的，因此，无论是哪台服务器为客户端分配的 sessionID，都务必保证全局唯一。

### 心跳检测机制

机器间的心跳检测机制是指在分布式环境中，不同机器（或进程）之间需要检测到彼此是否在正常运行，例如A机器需要知道B机器是否正常运行。在传统的开发中，我们通常是通过主机直接是否可以相互PING通来判断，更复杂一点的话，则会通过在机器之间建立长连接，通过TCP连接固有的心跳检测机制来实现上层机器的心跳检测，这些都是非常常见的心跳检测方法。

下面来看看如何使用ZK来实现分布式机器（进程）间的心跳检测。
基于ZK的临时节点的特性，可以让不同的进程都在ZK的一个指定节点下创建临时子节点，不同的进程直接可以根据这个临时子节点来判断对应的进程是否存活。通过这种方式，检测和被检测系统直接并不需要直接相关联，而是通过ZK上的某个节点进行关联，大大减少了系统耦合。

### Watcher监听事件

1. 监听的 Znode 被创建、删除、版本后变更(或数据变更)、子节点发生变更 会触发事件。
2. Watcher 是一次性的，一旦触发将会永久失效，如果需要反复监听就需要反复注册。

### Znode节点

在 zookeeper 中，“节点"分为两类，第一类同样是指构成集群的机器，我们称之为机器节点；第二类则是指数据模型中的数据单元，我们称之为数据节点一一ZNode。

* 持久节点：一旦创建将一直存在于服务端，除非客户端删除。
* 持久顺序节点： 在持久节点基础上，通过节点路径后缀一串序号来区分多个子节点创建的先后顺序。
* 临时节点：生命周期与客户端会话保持一致。
* 临时顺序节点：

### Stat节点状态

zookeeper 的每个 ZNode 上都会存储数据，对应于每个 ZNode，zookeeper 都会为其维护一个叫作 Stat 的数据结构，Stat 中记录了这个 ZNode 的三个数据版本：
* version（当前ZNode的版本）
* cversion（当前ZNode子节点的版本）
* cversion（当前ZNode的ACL版本）

### ACL权限控制

zookeeper 有以下五种权限：
* CREATE：创建子节点的权限。
* READ：获取子节点和子节点列表的权限。
* WRITE：更新节点数据的权限。
* DELETE：删除子节点的权限。
* ADMIN：设置节点ACL的权限。
其中需要注意的是，CREATE 和 DELETE 这两种权限都是针对**子节点**的权限控制。

## zookeeper支持的协议

### ZAB协议 & Paxos算法

Paxos 算法应该可以说是  zookeeper 的灵魂了。但是，zookeeper 并没有完全采用 Paxos算法 ，而是使用 ZAB 协议作为其保证数据一致性的核心算法。另外，在zookeeper的官方文档中也指出，ZAB协议并不像 Paxos 算法那样，是一种通用的分布式一致性算法，它是一种特别为 zookeeper 设计的崩溃可恢复的原子消息广播算法。

### ZAB协议

ZAB（ZooKeeper Atomic Broadcast 原子广播） 协议是为分布式协调服务 zookeeper 专门设计的一种支持崩溃恢复的原子广播协议。 在 zookeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议，zookeeper 实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性。

### ZAB协议的崩溃恢复和原子广播

在整个服务框架在启动过程中，或是当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进入恢复模式并选举产生新的 Leader 服务器。当选举产生了新的 Leader 服务器，同时集群中已经有过半的机器与该Leader服务器完成了状态同步之后，ZAB协议就会退出恢复模式。其中，所谓的状态同步是指数据同步，用来保证集群中存在过半的机器能够和Leader服务器的数据状态保持一致。

当集群中已经有过半的 Follower 服务器完成了和 Leader 服务器的状态同步，那么整个服务框架就可以进人消息广播模式了。 当一台同样遵守ZAB协议的服务器启动后加人到集群中时，如果此时集群中已经存在一个 Leader 服务器在负责进行消息广播，那么新加人的服务器就会自觉地进人数据恢复模式：找到 Leader 所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

## zookeeper集群

### 集群角色

* leader：负责进行投票的发起和决议，更新系统状态。
* follower：参与leader选举投票或事务请求投票，处理读请求 和 转发写请求给 leader。
* observer：弱化版的follower，不参与投票，只同步 leader 的状态，提高读取速度。

### 单机模式的安装

* Step1：配置JAVA环境，检验环境：java -version
* Step2：下载并解压zookeeper
```xml
cd /usr/local
wget http://mirror.bit.edu.cn/apache/zookeeper/stable/zookeeper-3.4.12.tar.gz
tar -zxvf zookeeper-3.4.12.tar.gz
cd zookeeper-3.4.12
```
* Step3：重命名配置文件zoo_sample.cfg
```xml
cp conf/zoo_sample.cfg conf/zoo.cfg
```
* Step4：启动zookeeper
```xml
bin/zkServer.sh start
```
* Step5：检测是否成功启动，用zookeeper客户端连接下服务端
```xml
bin/zkCli.sh
```

### 集群模式的搭建

本例搭建的是伪集群模式，即一台机器上启动三个zookeeper实例组成集群，真正的集群模式无非就是实例IP地址不同，搭建方法没有区别
Step1：配置JAVA环境，检验环境：java -version
Step2：下载并解压zookeeper
```xml
# cd /usr/local
# wget http://mirror.bit.edu.cn/apache/zookeeper/stable/zookeeper-3.4.12.tar.gz
# tar -zxvf zookeeper-3.4.12.tar.gz
# cd zookeeper-3.4.12
```
Step3：重命名 zoo_sample.cfg文件
```xml
# cp conf/zoo_sample.cfg conf/zoo-1.cfg
```
Step4：修改配置文件zoo-1.cfg，原配置文件里有的，修改成下面的值，没有的则加上
```xml
# vim conf/zoo-1.cfg
```

dataDir=/tmp/zookeeper-1
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890

Step5：再从zoo-1.cfg复制两个配置文件zoo-2.cfg和zoo-3.cfg，只需修改dataDir和clientPort不同即可
```
# cp conf/zoo-1.cfg conf/zoo-2.cfg
# cp conf/zoo-1.cfg conf/zoo-3.cfg
# vim conf/zoo-2.cfg
```
dataDir=/tmp/zookeeper-2
clientPort=2182

```
# vim conf/zoo-2.cfg
```
dataDir=/tmp/zookeeper-3
clientPort=2183

Step6：标识Server ID
创建三个文件夹/tmp/zookeeper-1，/tmp/zookeeper-2，/tmp/zookeeper-2，在每个目录中创建文件myid 文件，写入当前实例的server id，即1.2.3

```xml
# cd /tmp/zookeeper-1
# vim myid
1
# cd /tmp/zookeeper-2
# vim myid
2
# cd /tmp/zookeeper-3
# vim myid
3
```

Step7：启动三个zookeeper实例
```xml
# bin/zkServer.sh start conf/zoo-1.cfg
# bin/zkServer.sh start conf/zoo-2.cfg
# bin/zkServer.sh start conf/zoo-3.cfg
```
### 集群中leader的选举

选举场景：1、集群刚启动时   2、leader退出

1. 第一次每个 follower 都会选自己为 leader 服务器，也就是投出的是自己的服务器 ID 和 ZXID。
2. 每个 follower 都会受到来自其他 follower 的信息，并先按 ZXID 再按服务器 ID 最大选择选出新的选票，再次发出去。
3. 若某台服务器得到超半数的选票将当选为新的 leader。

### 集群数据同步流程

完成 leader 选举后，leader 等待 follower 来连接进行同步
1. leader 等待 follower 连接。
2. follower 将最大的 zxid 发送给 leader。
3. leader 根据 zxid 确定同步点。
4. 完成同步后，leader 给 follower 发送 uptodate 消息。
5. follower 收到 uptodate 后，又可以重新接受 client 连接请求了。

### 集群中事务请求处理流程

1. 所有事务请求都交由 leader 服务器来处理，leader 服务器会将一个事务请求转为一个proposal，并为其生成一个ZXID(事务ID)。
2. 之后 leader 服务器会将 proposal 放入每个 follower 的队列中（leader会为每个follower分配一个队列），并发送给 follower。
3. follower收到 proposal 后，会将事务日志写入磁盘，并在成功后返回 leader 一个ACK。
4. leader 只要收到过半的 follower 的ACK响应，就会广播一个 commit 消息给 follower 通知进行 proposal 的提交，同时自身也会完成。



## zookeeper的应用场景

### 实现命名服务(Name Service)

命名服务也是分布式系统中比较常见的一类场景。在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址，提供者等信息。被命名的实体通常可以是集群中的机器，提供的服务，远程对象等等——这些我们都可以统称他们为名字（Name）。

其中较为常见的就是一些分布式服务框架（如RPC、RMI）中的服务地址列表。通过在 zookeeper里创建顺序节点，能够很容易创建一个全局唯一的路径，这个路径就可以作为一个名字。

zookeeper 的命名服务即生成全局唯一的ID。

### 实现分布式协调/通知

zookeeper 中特有 Watcher 注册与异步通知机制，能够很好的实现分布式环境下不同机器，甚至不同系统之间的通知与协调，从而实现对数据变更的实时处理。使用方法通常是不同的客户端都对ZK上同一个 ZNode 进行注册，监听 ZNode 的变化（包括ZNode本身内容及子节点的），如果 ZNode 发生了变化，那么所有订阅的客户端都能够接收到相应的Watcher通知，并做出相应的处理。

ZK的分布式协调/通知，是一种通用的分布式系统机器间的通信方式。

### 实现分布式锁

有两种实现：
1. 集群中所有机器都去竞争创建某个节点，创建成功的机器相当于获取了这个锁。
2. 所有机器都去某个父节点下创建子节点，序号最小的机器获取锁。

### 实现分布式队列

1. 同步队列：在指定目录下创建 watcher，当监控到子节点数目达到指定值后再开始使用，进行读取消费。
2. FIFO队列：读取序列号最小的节点进行消费，例如实现生产者和消费者模型。
在此场景下，znode 节点值存储的可能就是消息本身。
