---
date: 2013-06-30
layout: post
title: Java并发中正确使用volatile
description: Use volatile for safe publication
categories:
- Blog
tags:
- Concurrency

---


前几天并发编程群里有同学对volatile的用法提出了疑问，刚好我记得Twitter有关实时搜索的这个[PPT](http://2011.lucene-eurocon.org/attachments/0002/8787/Busch_twitter_realtime_search_eurocon_11.pdf)对这个问题解释的很清晰并有一个实际的应用场景，于是周末把这个问题摘录了一些和并发相关的内容如下：

{:toc}

# 并发 - 定义
## 悲观锁 - Pressimistic locking

1. 一个线性在执行一个操作时持有对一个资源的独占锁。（互斥）
2. 一般用在冲突比较可能发生的场景下

## 乐观锁 - Optimistic locking

1. 尝试采用原子操作，而不需要持有锁；冲突可被检测，如果发生冲突，具有相应的重试逻辑
2. 通常用在冲突较少发生的场景下

## 非阻塞算法 - Non-blocking algorithm

1. 算法确保对线程间竞争共享资源时候，不会因为互斥而使任一线程的执行无限延迟；

## 无锁算法 - Lock-free algorithm

1. 如果系统整个流程的执行是无阻塞的(系统某一部分可能被短暂阻塞)，这种非阻塞算法就是无锁的。
2. 无锁算法比传统的基于锁的算法对系统的开销更小，且更容易在多核多CPU处理器上扩展；
3. 在实时系统中可以避免锁带来的延迟；
4. CAS (compare and swap)或LL/SC(load linked/store conditional)，以及内存屏障相关的指令经常被用在算法实现中。

## 无等待算法 - Wait-free algorithm

1. 如果每个线程的执行都是无阻塞的，这种非阻塞算法就是无等待的（比无锁算法更好）

# Java的并发

1. Java的内存模型并不保证一个线程可以一直以程序执行的顺序看到另一个线程对变量的修改，除非两个线程都跨越了同一个内存屏障。（Safe publication）

# Java内存模型

## 代码顺序规则

1. 一个线程内的每个动作 happens-before 同一个线程内在代码顺序上在其后的所有动作

## volatile变量规则

1. 对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入

## 传递性

1. 如果A happens-before B, B happens-before C，那 A happens-before C

# Safe publication案例

```
class VolatileExample {
    int x = 0;
    volatile int b = 0;

    private void write() {
        x = 5;
        b = 1;
    }

    private void read() {
        int dummy = b;
        while (x!=5) {
        }
    }

    public static void main(String[] args) throws Exception {
        final VolatileExample example = new VolatileExample();
        Thread thread1 = new Thread(new Runnable() {
            public void run() {
                example.write();
            }
        });

        Thread thread2 = new Thread(new Runnable() {
            public void run() {
                example.read();
            }
        });
        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
    }
}
```

<img src="https://www.evernote.com/shard/s26/sh/569ae14f-957d-4ca0-97d7-72f66479d298/b4fb554c924ec7c2356d3ca94d7be61a/deep/0/Screenshot%206/30/13%2010:48%20AM.png"/>

x并不需要定义为`volatile`, 程序里可以有需要类似x的变量，我们只需要一个volatile变量b来确保线程a能看到线程1对x的修改：

1. 根据代码顺序规则，线程1的`x=5;` happens-before `b=1;`; 线程2的`int dummy = b;`  happens-before `while(x!=5);`
2. 根据volatile变量规则，线程2的`b=1;` happens-before `int dummy=b;`
3. 根据传递性，`x=5;` happens-before `while(x!=5);` 

# JSR-133
在JSR-133之前的旧Java内存模型中，虽然不允许volatile变量之间重排序，但旧的Java内存模型仍然会允许volatile变量与普通变量之间重排序。JSR-133则增强了volatile的内存语义：严格限制编译器（在编译器）和处理器（在运行期）对volatile变量与普通变量的重排序，确保volatile的写-读和监视器的释放-获取一样，具有相同的内存语义。

延伸阅读： [JSR-133: JavaTM Memory Model and Thread Specification](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)， [The JSR-133 Cookbook for Compiler Writers](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)


# 参考链接

1. http://2011.lucene-eurocon.org/attachments/0002/8787/Busch_twitter_realtime_search_eurocon_11.pdf
2. http://www.rossbencina.com/code/lockfree
3. http://rethinkdb.com/blog/lock-free-vs-wait-free-concurrency/
4. http://www.infoq.com/cn/articles/java-memory-model-4
4. [JSR-133: JavaTM Memory Model and Thread Specification](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)
4. [The JSR-133 Cookbook for Compiler Writers](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)