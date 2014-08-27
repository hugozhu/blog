---
date: 2014-07-27
layout: post
title: 简单手机应用同步协议设计和实现
description: Sync data between mobile phone and server
categories:
- Blog
tags:
- Android

---

在手机上我们需要持久化应用的一些数据(典型的如本地的设置信息)，同时又希望能重装应用或换一台手机登录后能把这些数据再同步回来。业界有`syncml`标准，覆盖的功能很完善，正因为要保证兼容性，开源的实现都较重。如何借鉴这个标准自己来实现一个多端双向同步可扩展的功能呢？

我们假设一些前提：

1. 同一时刻只有一端（iPhone，iPad或其他移动设备）能和服务器同步；
2. ~~客户端和服务端的时间一致或误差较小；可在长连建立时通过协议计算时间差~~
3. 客户端保存全量数据；

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
1. LCID （Local Unique Identifiers）客户端生成的ID
2. GUID （Global Unique Identifiers）服务端生成
3. Anchor 锚点

## 客户端表设计

每条记录包含两个同步用的字段：
  
  *status* － 用来标识记录的状态：0－未同步（本地新增或更新）1 - 已同步，2 － 标记删除
  
  *anchor* － 用来记录服务端同步过来的时间戳。


## 双向同步流程示例

### 1. Client 增加2条记录

| ID | KEY  | VALUE | STATUS  | CREATED | MODIFIED| ANCHOR|
| ---| -----|-------| ------- | --------| ------- | ------|
| 1  | Foo  | Bar   |    0    |    1    |   1     |   0   |
| 2  | Hello| World |    0    |    2    |   2     |   0   |

### 2. Client 修改1条记录

| ID | KEY  | VALUE | STATUS  | CREATED | MODIFIED| ANCHOR|
| ---| -----|-------| ------- | --------| ------  | ------|
| 1  | Foo  | Bar   |    0    |    1    |   1     |   0   |
| 2  | Hello| World2|    0    |    2    |   3     |   0   |

### 3. Client 发送本地更新
`SELECT * FROM table WHERE STATUS ＝ 0 ORDER BY MODIFIED ASC` 会找出本地更新的所有记录，通过网络分批串行发送给服务端，一个请求可能包含多条记录，只有上一个请求得到响应后才能发起下一个请求。

### 4. Server 处理同步消息

服务端收到请求后根据记录是ADD，UPDATE或DELETE后在服务端数据库中做响应处理，处理时候需要比较客户端的ANCHOR（其实就是上一次同步服务端的MODIFIED）和服务端的记录MODIFIED，只有MODIFIED<ANCHOR的才能被更新，防止服务端有更加新的记录被覆盖。

`UPDATE table SET KEY = ?, VALUE = ? WHERE ID=? AND MODIFIED<=ANCHOR`

更新后服务端数据表如下：

| ID | KEY  | VALUE  | CREATED | MODIFIED|
| ---| -----|--------| --------| --------|
| 1  | Foo  | Bar    |    3    |   3     |
| 2  | Hello| World2 |    4    |   4     |

处理完成后响应请求数据如下，使用服务端Modified时间作为Anchor：

| ID | STATUS | ANCHOR|
| ---|  ------| ------|
| 1  |  1     |   3   |
| 2  |  1     |   4   |

### 5. Client 根据响应更新本地记录

`UPDATE table SET ANCHOR=X WHERE ID=? AND MODIFIED=?` 限制只有本地MODIFIED没修改过的记录才会更新ANCHOR，防止在同步期间更新过的本地数据被标记已同步。

| ID | KEY  | VALUE | STATUS  | CREATED | MODIFIED| ANCHOR|
| ---| -----|-------| ------- | --------| ------- | ------|
| 1  | Foo  | Bar   |    1    |    1    |   1     |   3   |
| 2  | Hello| World2|    1    |    2    |   2     |   4   |

### 6. Client 根据最大的ANCHOR请求 SERVER 增量同步

假设另一个客户端在服务器上增加了一条新的纪录并更新了一条记录：

| ID | KEY  | VALUE | CREATED | MODIFIED|
| ---| -----|-------| --------| ------- |
| 1  | Foo  | Bar2  |    3    |   9     |
| 2  | Hello| World3|    4    |   4     |
| 3  | Java | Bean  |    10   |   10    |

服务端执行`SELECT * FROM table WHERE MODIFIED > 4 ORDER BY MODIFIED ASC` 来根据客户端的ANCHOR=4获取未同步的数据, 并通过网络分批串行发给客户端

| ID | STATUS | ANCHOR|
| ---|  ------| ------|
| 1  |  1     |   9   |
| 3  |  1     |   10  |

### 7. Client 处理同步消息

客户端根据增量消息更新本地表

| ID | KEY  | VALUE | STATUS  | CREATED | MODIFIED| ANCHOR|
| ---| -----|-------| ------- | --------| ------- | ------|
| 1  | Foo  | Bar2  |    1    |    1    |   12    |   9   |
| 2  | Hello| World2|    1    |    2    |   2     |   4   |
| 3  | Java | Bean  |    1    |    12   |   12    |   10  |


















