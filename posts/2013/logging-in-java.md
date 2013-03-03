---
date: 2013-02-28
layout: post
title: Java程序的日志
description: Java日志最佳实践，给程序调试提供有用信息的日志
categories:
- Blog
tags:
- Java
- 代码之美

---

## Overview

一个在生产环境里运行的程序如果没有日志是很让维护者提心吊胆的，有太多杂乱又无意义的日志也是令人伤神。程序出现问题时候，从日志里如果发现不了问题可能的原因是很令人受挫的。本文想讨论的是如何在Java程序里写好日志。大多数的Web服务器（如Apache，Nginx）都有access日志和error日志，分别记录在不同的文件内；我们使用的服务器操作系统Linux有Syslog日志, /var/log目录下也有很多基础应用和服务的日志文件；桌面Windows有事件查看器, Mac有Console应用可以查看和管理日志；这些成熟的系统及工具方法都值得我们学习并在自己的项目中应用。

一般来说日志分为两种：业务日志和异常日志。使用日志我们希望能达到以下目标：

1.  对程序运行情况的记录和监控；
2.  在必要时可详细了解程序内部的运行状态；
3.  对系统性能的影响尽量小；


## 日志规范

程序框架应该提供统一的日志记录接口，日志格式也需要有一定的规范，方便利用日志工具来分析日志。
首先我们有必要了解一下Linux普遍使用的[Syslog](http://en.wikipedia.org/wiki/Syslog)标准协议，协议规定日志中应包含产生日志的模块(Facility)，严重性（Severity Level），时间，主机名或IP，进程名，进程ID和日志内容，根据模块和严重性可以配置相应的动作：是否需要记录，日志存储路径（文件或网络）。

下面是部分常见的Syslog模块类型：

**模块ID**    | **关键词** | **描述**
------------ | ------------- | ------------
0            | kern          | 内核消息 
1            | user          | 用户级别消息
2            | mail          | 邮件系统
3            | daemon        | 系统后台守护程序
4            | auth          | 安全/鉴权消息
5            | syslog        | syslogd内部产生的日志消息
…|           


以及Syslog严重程度划分：

**代码**      | **严重程度**   | **关键词**   | **描述**
------------ | ------------- | ------------| --------
0            | Emergency     | emerg(panic)| 紧急，系统已经不稳定了
1            | Alert         | alert       | 需要立刻采取措施
2            | Critical      | crit        | 严重情况
3            | Error         | err (error) | 系统出错
4            | Warning       | warning(warn)| 系统警告
5            | Notice        | notice       | 系统仍然正常，但值得注意
6            | Informational | info         | 正常系统通告
7            | Debug         | debug        | 系统调试信息

在你的Java程序里日志也可以参考Syslog的设计，根据业务对程序的模块和日志级别做一定的规划和设计。

## Java日志框架
Java的日志框架太多了。。。

1. [**Log4j**](http://logging.apache.org) 或 [**Log4j 2**](http://logging.apache.org/log4j/2.x/) - Apache的开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件、甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；用户也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，用户能够更加细致地控制日志的生成过程。这些可以通过一个配置文件（XML或Properties文件）来灵活地进行配置，而不需要修改程序代码。Log4j 2则是前任的一个升级，参考了Logback的许多特性；
2. [**Logback**](http://logback.qos.ch) - Logback是由log4j创始人设计的又一个开源日记组件。logback当前分成三个模块：logback-core,logback- classic和logback-access。logback-core是其它两个模块的基础模块。logback-classic是log4j的一个改良版本。此外logback-classic完整实现SLF4J API使你可以很方便地更换成其它日记系统如log4j或JDK14 Logging；
3. [**java.util.logging**](http://docs.oracle.com/javase/6/docs/api/java/util/logging/package-summary.html) - JDK内置的日志接口和实现，功能比较简单...；
4. [**Slf4j**](http://www.slf4j.org) - SLF4J是为各种Logging API提供一个简单统一的接口），从而使用户能够在部署的时候配置自己希望的Logging API实现；
5. [**Apache Commons Logging**](http://commons.apache.org/proper/commons-logging/) - Apache Commons Logging （JCL）希望解决的问题和Slf4j类似。


选项太多了的后果就是选择困难症，我的看法是没有最好的，只有最合适的。在比较关注性能的地方，选择Logback或自己实现高性能Logging API可能更合适；在已经使用了Log4j的项目中，如果没有发现问题，继续使用可能是更合适的方式；我一般会在项目里选择使用Slf4j, 如果不想有依赖则使用java.util.logging或框架容器已经提供的日志接口。

## Java日志最佳实践

### 定义日志变量

需要定义成final static

### 日志分级
Java的日志框架一般会提供以下日志级别，缺省打开info级别，也就是debug，trace级别的日志在生产环境不会输出，在开发和测试环境可以通过不同的日志配置文件打开debug级别。

1. **fatal** - 严重的，造成服务中断的错误；
2. **error** - 其他错误运行期错误；
3. **warn** -  警告信息，如程序调用了一个即将作废的接口，接口的不当使用，运行状态不是期望的但仍可继续处理等；
4. **info** -  有意义的事件信息，如程序启动，关闭事件，收到请求事件等；
5. **debug** - 调试信息，可记录详细的业务处理到哪一步了，以及当前的变量状态；
6. **trace** - 更详细的跟踪信息；

在程序里要合理使用日志分级:
    
    //调试的时候可以知道进入了方法
    logger.debug("entering getting content");
    String content = CacheManager.getCachedContent();
    if (content == null) {
        
        //使用warn，因为程序还可以继续执行，但类似警告太多可能说明缓存服务不可用了，值得引起注意
        logger.warn("Got empty content from cache, need perform database lookup"); 
        
        Connection conn = ConnectionFactory.getConnection();
        if (conn == null) {
            logger.error("Can't get database connection, failed to return content"); //尽量提供详细的信息，知道错误的原因，而不能简单的写logger.error("failed")
        } else {
            try {
                content = conn.query(...);
            } catch (IOException e) {
                //异常要记录错误堆栈
                logger.error("Failed to perform database lookup", e);
            } finally {
                ConnectionFactory.releaseConnection(conn);
            }
        }
    }
    //调试的时候可以知道方法返回了
    logger.debug("returning content: "+ content);
    return content;

上面这段示范代码演示了各种级别的使用，但其中有个问题是debug日志太多后可能会影响性能？有一种改进方法是：

    if (logger.isDebugEnabled()) {
        logger.debug("returning content: "+ content);
    }

但更好的方法是Slf4j提供的[最佳实践](http://www.slf4j.org/faq.html#logging_performance):

```
logger.debug("returning content: {}", content);
```
一方面可以减少参数构造的开销，另一方面也不用多写两行代码；    

### 有意义的日志
通常情况下在程序日志里记录一些比较有意义的状态数据：

1. 程序启动，退出的时间点；
2. 程序运行消耗时间；

    ```        
    long startTime = System.currentTime();          
    // business logical          
    logger.info("execution cost : " + (System.currentTime() - startTime) + "ms");　      
    ```
3. 耗时程序的执行进度，不然程序开始运行后半天没一点输出挺让人着急啊~
4. 重要变量的状态变化。

### 日志安全
日志中尽量不要包含敏感信息，对于敏感信息如用户身份证号码，密码可以加密后存储；以防止日志文件不慎外泄时保全用户的数据安全；日志通常不允许修改，必要时还可以通过校验位来鉴别日志是否正确。

### 日志可靠性
TODO：

### 日志监控方法
TODO：

### 作业
TODO：

#### 参考链接

1. http://www.slf4j.org/manual.html
2. http://commons.apache.org/proper/commons-logging//guide.html#JCL_Best_Practices
3. http://wikipedia.org/wiki/syslog 