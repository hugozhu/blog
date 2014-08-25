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

在手机上我们需要持久化应用的一些数据(典型的如本地设置信息)，同时又希望能重装应用或换一台手机登录后能把这些数据再同步回来。

## 应用场景
1. 通讯录同步
2. 最近联系人
3. App客户端设置
4. 最近会话列表
5. 黑名单
6. 群设置
7. 用户设置

## ID处理
1. 客户端生成LCID （Local Unique Identifiers）
2. 服务端生成GUID （Global Unique Identifiers）

## 修改检测

## 双向同步流程
1. 客户端发起同步请求：data_type, last_anchor
2. 如果last_anchor为0，服务端返回：sync_session_id，meta info
3. ...
