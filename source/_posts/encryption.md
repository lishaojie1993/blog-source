---
title: 加密算法
date: 2019-07-03 23:21:25
categories: 
  - Java
  - 算法
tags:
  - 算法
  - 加密算法
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## 加密算法的种类

从宏观上来看，加密算法可以归结为三大类：**哈希算法、对称加密算法、非对称加密算法。**

## 哈希算法

从严格意义上来说，**哈希算法并不属于加密算法**，但它在信息安全领域也起到了很重要的作用。其中一个重要的作用就是**生成信息摘要**，用以验证原信息的完整性和来源的可靠性。

举个例子，在网上买东西，需要用到支付宝付款，于是付款时需要通知支付宝，并告诉支付宝商户ID、支付金额等等信息。具体过程如下：（假如key=abc，**Hash（1234_100_abc） = 948569CD3466451F**）

<!-- More -->

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gazjr47fjuj30fp08d0t4.jpg)

请求方把所有参数，外加双方约定的key拼接起来，并利用哈希算法生成一段信息摘要，而接收方在接收到参数和摘要后，按照同样的规则，也把参数和key拼接起来生成摘要并进行比较，如果完全一致，则证明信息没有被篡改。

生成信息摘要的过程叫做**签名**，验证信息摘要的过程叫做**验签**。

哈希算法最著名的当属**MD5算法**。后来，人们觉得MD5算法生成的信息摘要太短（128位），不够安全，于是又有了**SHA系列算法**。

------

## 对称加密算法

上面提到的哈希算法可以解决验签问题，却不能解决明文加密问题。

### 什么是对称加密？

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gazjwvskkzj30gw05c3z0.jpg)

如图所示，一段明文通过密钥进行加密，可以生成一段密文；这段密文通过同样的密钥进行解密，可以还原成明文。这样一来，只要双方事先约定好了密钥，就可以使用密文进行往来通信。

除了通信过程中的加密以外，数据库存储的敏感信息也可以通过这种方式进行加密。这样即使数据泄露到了外界，泄露出去的也都是密文。

### 对称加密包含哪些算法？

在早期，人们使用**DES算法**进行加密解密；后来，人们觉得DES不够安全，发明了**3DES**算法；而如今，最为流行的对称加密算法是**AES算法**。

### 对称加密的优缺点

对称算法的好处是加密解密的效率比较高，缺点是不够安全，因为通信双方约定的密钥是相同的，只要密钥本身被任何一方泄露出去，通信的密文就会被破解；此外，在双方建立通信之初，服务端把密钥告诉给客户端的时候，也有被拦截到的危险。

------

## 非对称加密算法

### 什么是非对称加密？

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gazk870sktj30ed09kmyf.jpg)

如图所示，在非对称加密中存在一对密钥，一个叫做**公钥**，另一个叫做**私钥**。在加密解密的过程中，我们既可以使用公钥加密明文，使用私钥解密密文；也可以使用私钥加密明文，使用公钥解密密文。其中最著名的非对称加密当属**RSA算法**。

### 非对称加密的通信过程

1. 在双方建立通信的时候，服务端只把公钥发送给客户端，自己保留私钥。
2. 客户端利用接受到的公钥，加密另外一个密钥X（可以是对称加密的密钥），发送给服务端。
3. 服务端获得消息后，利用自己的私钥解密，得到里面隐含的密钥X。
4. 从此以后，双方可以利用密钥X进行对称加密的通信了。

### 非对称加密的优缺点

好处就是安全性很高，在通信过程中，即使公钥被第三方截获，甚至后续的所有通信都被截获，第三方也无法进行破解。因为第二步利用公钥加密的消息，只有私钥才能解开，所以第三方永远无法知道密钥X是什么。

缺点是性能较差，无法应用于长期的通信。