---
date: 2013-03-01
layout: post
title: Java的资源管理
categories:
- Blog
tags:
- Java
- 代码之美

---

## Overview

Java程序中的常见的资源有：文件，Socket，数据库连接。在使用这些资源时候要分外小心，因为操作系统可同时操作的资源是有限的，比如默认情况下系统允许同时打开的文件数为1024个，Mysql服务器默认允许的最大连接数是100，所以操作这些资源时候要注意即使在遇到错误时也要让系统能正确回收资源。如果发生错误时候，打开的文件描述符没关闭或数据库连接没关闭，积累到一定程度后，应用将会变得不可用，只能重启。


## try-catch-finally

Java提供了try-catch-finally来保证程序遇到异常时总是有机会可以处理资源的关闭 -- 调用资源对象的close()方法。但经验不足的Java程序员还是会错误的管理资源，而造成资源的泄露，静态代码分析工具，如[**FindBugs**](http://findbugs.sourceforge.net)可以帮助发现此类问题。

首先我们来看一段文件操作代码：

    private void copy(String from, String to) throws IOException {
        FileInputStream in = null;  
        FileOutputStream out = null;  
        in = new FileInputStream(from);  
        out = new FileOutputStream(to);  
        int c;  
        while ((c = in.read()) != -1)
            out.write(c);  
        in.close();
        out.close();
    }

一眼看上去，代码挺整齐的，逻辑也容易理解。但其中有一个很大的问题是，如果out.write调用失败（比如磁盘空间满了）方法异常退出，in.close()和out.close()就不会被调用，而in和out对象内部都引用了系统资源-[文件描述符](http://zh.wikipedia.org/wiki/文件描述符)，这样会导致文件描述符没有关闭，不能被重新使用而直到整个Java进程退出。

#### [File descriptor](http://en.wikipedia.org/wiki/File_descriptor)
Linux的每个进程（如：Java进程）都有一个文件描述符表管理当前进程访问的所有的文件，文件描述符关联了系统文件表中的file entry，系统能容纳多少file entry是有限制的，如果超过限制系统会拒绝访问，抛出Too many opened files错误。

<img src="https://pbs.twimg.com/media/BEUt3v3CEAAeP-m.jpg:large"/>

较为正确的代码应该是：

    private void copy(String src, String dest) throws IOException {
        FileInputStream in = null;  
        FileOutputStream out = null;  
        try {
            in = new FileInputStream(src);  
            out = new FileOutputStream(dest);  
            int c;  
            while ((c = in.read()) != -1)
                out.write(c);
        } finally {
             try {
                 if (in!=null) {
                    in.close();
                 }
             } finally {
                 if (out!=null) {
                    out.close();
                 }
             }
        }
    }


但是这样的代码写起来是不是让人有点沮丧？这样写代码犯错的可能性确实比较大。
改良过后的代码阅读性好一些：
       
    private void copy(String src, String dest) throws IOException {
        FileInputStream in = null;  
        FileOutputStream out = null;  
        try {
            in = new FileInputStream(src);  
            out = new FileOutputStream(dest);  
            int c;  
            while ((c = in.read()) != -1)
                out.write(c);  
        } finally {
            FileUtils.close(in);
            FileUtils.close(out);
        }
    }
    

让我们也看看在其他语言里是怎么实现的：

### Ruby：

    def copy(src, dest)
        File.open(dest, 'w') do |f|  
            f.write(File.read(src))
        end  
    end
    
Ruby的File.open 方法接受一个函数作为参数，执行该函数后，会保证打开的文件被关闭，即使在执行函数过程中有异常。相比之下这种代码优美多了有没有？

### Golang：

    func copy(src string, dest string) {
        src_file, err := os.Open(src)
        if err != nil { panic(err) }
        defer src_file.Close()
        
        dest_file, err := os.Open(dest)
        if err != nil { panic(err) }
        defer dest_file.Close() 
        
        buf := make([]byte,1024)
    
        for {
            n, err := src_file.Read(buf)
            if err != nil && err != io.EOF { panic(err) }
            if n == 0 {break}
            
            if _, err:= dest_file.Write(buf[:n]); err != nil {
               panic(err)
            } 
        }       
    }

Go语言通过defer关键词来保证程序结束时相应的方法会被调用，嗯，你还是要显示的写Close()方法，但有一点改进就是你可以在打开后立刻写关闭语句，只要加上defer关键词。

### Clojure：
    defn copy[src dest] ( 
       (with-open [rdr (reader src)
                   wrtr (writer dest)]
          (doseq [line (line-seq rdr)]
              (.write wrtr line))))

Clojure通过with-open函数来保证打开的文件在异常情况下也会被关闭

### Java 7：

    private void copy(String src, String dest) throws IOException {
        FileInputStream in = null;  
        FileOutputStream out = null;  
        try (FileInputStream in = new FileInputStream(src);  
             FileOutputStream out = new FileOutputStream(dest)) {            
            int c;  
            while ((c = in.read()) != -1)
                out.write(c);  
    }

终于Java 7通过自动资源管理较好的解决了这个问题，try()代码块内的变量在离开时候会自动调用AutoCloseable接口必须实现的close()方法。java.lang.AutoCloseable接口是在Java 7内新增的。
    

## 独占资源和并发访问

### 锁

#### 读写锁

### 信号量

### 连接池

### 作业
TODO：

#### 参考链接

1. [Better Resource Management with Java SE 7: Beyond Syntactic Sugar](http://www.oracle.com/technetwork/articles/java/trywithresources-401775.html)