---
date: 2014-08-27
layout: post
title: 简单手机应用同步协议设计和实现
description: Sync data between mobile phone and server
categories:
- Blog
tags:
- Android


---

{:toc}

在手机上我们需要持久化应用的一些数据(典型的如本地的设置信息)，同时又希望能重装应用或换一台手机登录后能把这些数据再同步回来。业界有[SyncMl](http://en.wikipedia.org/wiki/SyncML)标准，覆盖的功能很完善，正因为要保证兼容性，开源的实现都较重。如何借鉴这个标准自己来实现一个多端双向同步可扩展的功能呢？

App使用同步协议可以将原本必须在线操作的功能（如：删除一个联系人，修改一个联系人的备注信息）也可以在断网情况下完成。

我们假设一些前提：

1. 同一时刻只有一端（iPhone，iPad或其他移动设备）能和服务器同步；
2. ~~客户端和服务端的时间一致或误差较小；可在长连建立时通过协议计算时间差~~
3. 客户端保存全量数据，对于客户端只需要部分数据的方案需要修改；

## 应用场景
1. 通讯录同步
2. 最近联系人
3. App客户端设置
4. 最近会话列表
5. 黑名单
6. 群设置
7. 群成员
8. 用户的一些设置和开关

## 名词解释
1. LCID - Local Unique Identifiers，  客户端生成的记录ID，客户端唯一；
2. GUID - Global Unique Identifiers， 服务端生成的记录ID，全局唯一；
3. Anchor - 同步锚点。可以使用递增的序列号或时间戳来表示，用来发现两端数据变化的部分。

## 客户端表设计

每条记录包含两个同步用的字段：
  
  *status* － 用来标识记录的状态
  
|Status|  含义    |
| -----|----------|
| 0    |  本地新增 |
| -1   |  标记删除 |
| 1    |  本地更新 |
| 9    |  已同步   |
  
  *anchor* － 用来记录服务端同步过来的时间戳。

## 服务端表设计
	
  *modified* － 记录在服务端的修改时间
  

## 双向同步流程示例

### 1. Client 增加2条记录

| id | key  | value | status  | modified| anchor|
| ---| -----|-------| ------- | ------- | ------|
| 1  | Foo  | Bar   |    0    |   1     |   0   |
| 2  | Hello| World |    0    |   2     |   0   |

### 2. Client 修改1条记录

| id | key  | value | status  | modified| anchor|
| ---| -----|-------| ------- | ------  | ------|
| 1  | Foo  | Bar   |    0    |   1     |   0   |
| 2  | Hello| World2|    1    |   3     |   0   |

### 3. Client 发送本地更新
执行SQL `SELECT * FROM table WHERE status < 9 ORDER BY modified ASC` 找出客户端本地需要同步到服务端的记录，通过网络分批串行发送给服务端，一个请求可能包含多条记录，只有上一个请求得到响应后才能发起下一个请求。

发送的同步消息需包含ANCHOR：

| id | key  | value | status  | modified| anchor|
| ---| -----|-------| ------- |---------|-------|
| 1  | Foo  | Bar   |    0    |  1      |  0    |
| 2  | Hello| World2|    1    |  3      |  0    |

### 4. Server 处理同步消息

服务端收到请求后先比较客户端的`anchor`（其实就是上一次同步服务端的`modified`）和服务端的记录MODIFIED，只有服务端`modified`=客户端`anchor`的才能继续同步，否则说明客户端在上一次同步后服务端又更行了，需要解决冲突后再继续(见:`冲突解决1`)；继续同步的记录则根据记录状态做ADD，UPDATE或DELETE操作。

`REPLACE INTO table (id, key, value) VALUES (?, ?, ?)`

更新后服务端数据表如下：

| id | key  | value  | modified|
| ---| -----|--------|---------|
| 1  | Foo  | Bar    |  3      |
| 2  | Hello| World2 |  4      |

请求响应数据如下(使用了服务端`modified`时间作为`anchor`)：

| id | status | anchor|
| ---| -------| ------|
| 1  |  9     |   3   |
| 2  |  9     |   4   |

### 5. Client 根据响应更新本地记录

得到服务端的同步消息响应后，客户端应判断发出同步请求时的`modified`和此时本地记录的`modified`是否一致，防止在同步期间本地数据被用户再次更新。

相应的SQL如下：
`UPDATE table SET status=9, anchor=?, modified=? WHERE id=? and modified=?`

| id | key  | value | status  | modified| anchor|
| ---| -----|-------| ------- | --------|-------|
| 1  | Foo  | Bar   |    9    |   1     | 0->3  |
| 2  | Hello| World2|    9    |   2     | 0->4  |

### 6. Client 请求 Server 增量同步
因为服务端的`modified`是随着时间递增的。客户端本地保存的记录中最大的anchor：`Max(anchor)`可以认为是上一次同步的时间。如果服务端有比`Max(anchor)`更大的`modified`的记录, 则需要向客户端同步。

假设另一个客户端在服务器上增加了一条新的纪录并更新了一条记录：

| id | key  | value | modified|
| ---| -----|-------| --------|
| 1  | Foo  | Bar2  |  3->9   |
| 2  | Hello| World3|  4->4   |
| 3  | Java | Bean  |  0->10  |

根据客户端的`Max(anchor)`=4，服务端执行`SELECT * FROM table WHERE modified > 4 ORDER BY modified ASC`来获取未同步的数据, 并通过网络分批串行发给客户端。

| id | status | anchor|
| ---|  ------| ------|
| 1  |  9     |   9   |
| 3  |  9     |   10  |

### 7. Client 处理同步消息

客户端根据增量消息更新本地表。处理消息时，只能更新状态为已同步或不存在的记录（也就是`status=9`），防止在增量同步过程中，客户端本地记录被用户修改。

| id | key  | value | status  | modified| anchor|
| ---| -----|-------| ------- | --------| ------|
| 1  | Foo  | Bar2  |    9    |   1-12  |   9   |
| 2  | Hello| World2|    9    |   2     |   4   |
| 3  | Java | Bean  |    9    |   12    |   10  |

### 8. Client 删除记录

客户端需要做逻辑删除，将status设成-1

| id | key  | value | status  | modified| anchor|
| ---| -----|-------| ------- | --------| ------|
| 1  | Foo  | Bar2  |    9    |   1-12  |   9   |
| 2  | Hello| World2|    -1   |   15    |   4   |
| 3  | Java | Bean  |    9    |   12    |   10  |

客户端同步引擎将则根据`status<9`将这条消息发送给服务端（注：如果anchor=0可以不发送），服务端按第四步同样的处理方法将服务端记录标记为删除，并给客户端同步成功的响应；客户端在得到响应后可以将本地记录物理删除。

### 9. 服务端 删除记录

如果增量下行到客户端的消息中包含了删除记录的消息，客户端可以直接物理删除。


# 冲突解决

## 1. 服务端解决客户端Anchor和记录Modified不一致的情况
有可能是手机A在上一次同步成功后更新了一条记录后断线造成更新未同步；过段时间后换了另一只手机B对同一条记录做了修改并同步成功；再切换回手机A时，继续同步时，服务端会发现A发送过来的ANCHOR较服务端MODIFIED小而拒绝这条记录的同步，并在响应内将服务端的记录返回给客户端。(以服务端记录为准)







