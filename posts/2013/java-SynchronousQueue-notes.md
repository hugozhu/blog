---
date: 2013-03-05
layout: post
title: Java并发包中的SynchronousQueue的使用
description: Notes about [SynchronousQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/SynchronousQueue.html) design
categories:
- Blog
tags:
- Java
- Concurrent Programming

---

## Overview

Java并发编程包中的[SynchronousQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/SynchronousQueue.html)实现了[BlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/BlockingQueue.html)的接口，一个线程（发布者）对其的插入操作必须等待另一个线程（消费者）的移除操作，反过来也一样。但不像ArrayBlockingQueue或LinkedListBlockingQueue，SynchronousQueue内部并没有任何数据缓存空间，你不能调用peek()方法来看队列中是否有数据元素，因为数据元素只有当你试着取走的时候才可能存在，不取走而只想偷窥一下是不行的，当然遍历这个队列的操作也是不允许的。队列头元素是第一个排队要插入数据的**线程**，而不是要交换的数据。数据是在生成者和消费者线程之间直接传递的，而不需要先缓冲到队列中。

从集合的角度来看，SynchronousQueue像是一个一直为空的集合，iterator()永远为空，size()方法永远返回0；

从功能的角度来看，`new SynchronousQueue()`则和`new ArrayBlockingQueue(1)`差不多。


## 实现原理
SynchronousQueue的实现采用了一种**无锁算法** -- 扩展的“[Dual stack and Dual queue](http://www.cs.rochester.edu/research/synchronization/pseudocode/duals.html)”算法。竞争机制支持公平和非公平两种：非公平竞争模式使用后进先出栈(Lifo Stack)数据结构；公平竞争模式使用先进先出队列（Fifo Queue），性能上两者是相似的，在常见应用里，Fifo通常可以支持更大写的吞吐量，但Lifo可以更大程度的保持线程的本地化。

代码实现里的双队列（包含了一个先进先出队列和一个先进后出队列）则是用来在保存“数据“的，其状态为以下三种情况：

1. put操作的元素
2. take请求的元素空间
3. 为空。


核心的接口是Transfer，双栈和双队列都实现了这个接口，生产者的put或消费者的take都使用这个接口，根据第一个参数来区别：

```
    /**
     * Shared internal API for dual stacks and queues.
     */
    static abstract class Transferer {
        /**
         * Performs a put or take.
         *
         * @param e if non-null, the item to be handed to a consumer;
         *          if null, requests that transfer return an item
         *          offered by producer.
         * @param timed if this operation should timeout
         * @param nanos the timeout, in nanoseconds
         * @return if non-null, the item provided or received; if null,
         *         the operation failed due to timeout or interrupt --
         *         the caller can distinguish which of these occurred
         *         by checking Thread.interrupted.
         */
        abstract Object transfer(Object e, boolean timed, long nanos);
    }
```
