---
title: Java学习
date: 2020-03-10 17:13:11
categories: [Java]
tags: [Java]
---
## 1、基本概念
### 1、JVM、JRE、以及JDK（以Windows为例）

* 1、JVM是JAVA虚拟机(java.exe)，是JAVA能够实现跨平台的关键部分；
* 2、JRE = JRE + rt.jar基础类库；
* 3、JDK是在JRE的基础上加上了一些程序开发所需要的程序及Jar包；如编译器（javac.exe）、开发工具（javadoc.exe、jar.exe、keytool.exe、jconsole.exe），Jar包包括dt.jar、tools.jar等
  
其中rt.jar包为所有核心Java 运行环境的已编译class文件的集合;tools.jar是系统用来编译一个类的时候用到的，即执行javac所需要用到的；dt.jar包是关于运行环境的类库，主要是swing包。

## 2、Java语法

### 1、集合

#### 1、接口

Java的集合类主要由两个接口派生而出：**Collection**和**Map**。

* Collection
![Collection接口及其派生类.png](../img/Collection接口及其派生类.png)
* Map
![Map接口及其派生类.png](../img/Map接口及其派生类.png)

#### 2、工具类

集合框架中的工具类，**Collections**与**Arrays**，这两个工具类中的方法都是静态的。

* Collections
用于操作集合。
* Arrays
用于操作数组。


#### 3、HashMap与HashTable
* HashMap  
  
HashMap的一些知识在面试中几乎是不可避免的考点，其中涉及到的知识点包括扩容机制（resize），底层实现等等。  
HashMap允许键值对都为null,并且是非线程安全的。
在JDK1.8之前，HashMap底层实现是数组+链表，在JDK1.8时，在原来基础上，变成了数组+（链表/红黑树）的存储方式，进一步提高性能；提高性能的原因是红黑树是平衡二叉树，在查找性能方面比链表高；链表变为红黑树的时机是当链表个数达到8的时候就将用链表存储改为红黑树存储；

HashMap有两个参数会影响其的扩容机制，一是初始容量，二是装载因子；当在add元素后，会判断map中元素数量超过 容量*装载因子时，map就会执行扩容（resize）操作，还有一个操作比较耗时，当链表转变为红黑树时，会判断此时数组的容量，如果数组容量小于64，不会进行链表转红色的操作，还是执行扩容操作。扩容比较耗时，日常开发中应该避免此操作，在map初始化时应该给与一个比较合适的初始容量，避免扩容；  
扩容是申请一个容量为原数组两倍的新数组，然后将数组元素进行复制。在jdk1.7及其之前需要对元素重新进行hash，在jdk1.8时

* HashTable  
  
  HashTable不允许值为null，如果置入null，会报空指针异常；
