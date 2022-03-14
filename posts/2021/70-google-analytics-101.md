---
date: 2022-03-10
layout: post
title: Google Analytics 101
description: How to use Google Analytics to measure shopping app
categories:
- Blog
tags:
- Google Analytics

---

{:toc}

# Google Analytics 4 介绍
Google Analytics（分析）可帮助您了解人们使用您的 Web、Apple 或 Android 应用的方式。SDK 会自动收集大量事件和用户属性，您也可以定义自定义事件，以便衡量对您的业务有特殊影响的因素。收集到数据后，可通过 Firebase 控制台到信息中心中查看。通过此信息中心，您可以深入、详细地了解您的数据，包括摘要数据（如活跃用户和受众特征）和更详细的数据（如识别您最畅销的商品）。

Analytics 还集成了 Firebase 的一些其他功能。例如，它自动记录与通过通知编辑器发送的通知消息相对应的事件，并就每个广告系列的影响提供报告。

Analytics 可帮助您了解用户的行为方式，以便您就如何推广您的应用制定明智的决策。您可以查看您的广告系列在自然渠道和付费渠道的效果，以了解哪些方法对于吸引高价值用户最为有效。如果您需要执行自定义分析或者将您的数据与其他源数据联接，您可以将自己的 Analytics 数据关联到 BigQuery，从而进行更复杂的分析，例如查询大型数据集以及联接多个数据源。

# 网站和App用户行为跟踪技术的演进
UA （Google Universal Analytics, GA3) -> GA4（Google  Analytics 4)
https://support.google.com/analytics/answer/10270783?hl=zh-Hans

GA4是UA（GA3）的一次全新的升级，测量模型从以传统的基于会话（Session）转变为事件（Event）驱动，GA3专注于在一次会话中收集和处理用户的各种行为数据（如页面展现，事件，交易），在GA4中，一次页面展现（Pageview）也被认为是一个事件，而页面的标题（Page Title）和路径（Page Path）则是事件的参数。

三种实现方案：analytics.js -> gtag.js -> GTM (Google Tag Manager)

analytics.js已不被推荐使用，GTM是目前的最佳实践。

GTM的三个优势：

* 可以填入第三方追踪代码，例如 Facebook/Tiktok Pixel
* 追踪事件 (转化) 或其他项目时候，不用工程师改代码
* 有代码预览，审核发布流程，适合团队协作

UA（GA3）只有一种跟踪代码类型，但GA4有两种代码类型：GA4配置代码和GA4事件代码，其中GA4配置代码的作用域范围是页面全局，用户在页面上所有交互事件共享相同的配置。运行期GA4配置代码要在事件代码触发前执行。

如果网站页面上既有gtag.js，又有GTM跟踪代码，理论上这并不会产生问题，但要注意GTM重复配置事件触发可能导致多次同一事件多次触发；另外，gtag.js的事件参数如 “cookie_prefix”, or “allow_ad_personalization_signals”会传递到GTM里配置的GA4事件。

# 重要概念

## Segments（受众群体）
建立起产品受众分层是产品精准营销的基础。受众群体的分层是产品用户的具体分类，可根据地区，访问设备，年龄等用户属性来分类，每一个产品都有一个目标获取的受众群体，具体的营销活动应该根据目标受众的喜好来追求ROI最大化，用户增长首先要找到最容易获取的受众群体。

Segment示例：

* (客层/年龄：18-24)「AND」(客层/性别：女性)
* 行为：工作阶段 > 1「AND」行为：每位使用者交易次数 > 1
* 电子商务：每位使用者收益 > 10「AND」电子商务：产品 = T恤

## Event（事件）
https://support.google.com/analytics/answer/9322688?hl=zh-Hans&ref_topic=9756175
用户在你的网站或App中通过互动而触发的各种事件参数的数据，比如当打开一个网页时会触发一个“page_view”事件。

GA4内置了近500种事件，包含了电商网站，游戏App，旅游网站，视频网站等常见的事件类型，其中增强型衡量事件需要通过配置打开来生效，推荐的事件需要工程师编写代码来实现（注：事件名称和事件参数和参数值的规范要符合GA4的数据模型）

* 自动收集的事件 https://support.google.com/analytics/answer/9234069?hl=zh-Hans&ref_topic=9756175

* 增强型衡量事件 https://support.google.com/analytics/answer/9216061?hl=zh-Hans&ref_topic=9756175

* 推荐的事件 https://support.google.com/analytics/answer/9267735?hl=zh-Hans&ref_topic=9756175

** 如果以上的事件和事件参数还无法满足跟踪需求，那么你还可以采用自定义事件（尽量少用）**

* 自定义事件 https://support.google.com/analytics/answer/11262438?hl=zh-Hans&ref_topic=9756175

## Dimension （维度）
通过用户在你的网站或App中触发的各种事件（Event）收集而来的数据的属性或特征，如UserId，日期，流量来源，用户所属地域，用户使用的设备等

## Metrics （指标）
指标是量化可衡量的，通过指标你可以知道什么事件发生了多少次，

## Conversion （转化）
Conversion是高价值的事件（Event），借助转化跟踪，你可以了解你投放的广告获得的点击是否有效转化为你网站上有价值的客户活动，例如购买、注册，喜欢，加购等，不同的事件有不同的价值。
https://support.google.com/google-ads/answer/6095821?hl=zh-Hans

## Attribution models (归因模型)
https://support.google.com/analytics/answer/1033861?hl=zh-Hans#AttributionModels&zippy=%2C%E6%9C%AC%E6%96%87%E5%8C%85%E5%90%AB%E7%9A%84%E4%B8%BB%E9%A2%98

为了解答有关用户行为的各种网站分析问题，Google Analytics（分析）会使用各种计算类型或归因模型来得出您在报告中看到的数据。请将每份 Google Analytics（分析）报告视为对某类用户分析问题的解答。通常，这些问题可以划分为以下几类：

* 内容：特定网页被浏览的次数。
* 目标：哪些网页网址对目标转化率的贡献最大。
* 电子商务：给定网页为交易贡献多少价值。
* 内部搜索：哪些内部搜索字词促成了交易。

对于以上各种主要类别及其包含的报告，Google Analytics（分析）会使用不同的归因模型。由于每种归因模型都是专为计算一组已知指标而设计的，您可能会注意到某些指标（例如，网页浏览量）只在某些报告中显示而不在其他报告中显示。这是由该报告所使用的归因模型决定的。

Google Analytics（分析）报告使用 3 种归因模型：

* 依据请求
* 网页价值
* 网站搜索归因

# GA4 和 GTM 事件跟踪使用流程
** 重要！！！请工程师阅读并完整完成一次流程，从GA4报表数据确认配置正确 ** 
https://www.optimizesmart.com/event-tracking-in-google-tag-manager-v2-complete-guide/

## DataLayer
用于传递Event数据给GTM的一个Javascript对象。
https://support.google.com/tagmanager/answer/6164391?hl=zh-Hans#:~:text=A%20data%20layer%20is%20a,developer%20documentation%20for%20more%20information.


## Set User Properties 设置用户属性
https://firebase.google.com/docs/analytics/user-properties?platform=android

## Logging Event 记录事件
https://firebase.google.com/docs/analytics/events?platform=android

```
以登录界面为例：

通用屏幕打点：
logEvent({
  name: "screen_view",
  params: {
    screen_name:  "login_view", //iOS和Android保持一致
    screen_title: "Login and Register"
  }
}

用户页面互动自定义打点：
logEvent({
  name: "hho_login_view",
  params: {
    click : "tab_signin | tab_register | btn_signin | btn_register | link_forget_password | link_bottom_privacy | link_bottom_service",
    input: "email ｜ password"
  }
})

在部署了gtag.js中页面中logEvent的实现如下：
function logEvent(name, params) {
  gtag("event", name, params)
}


在部署了GTM的页面中logEvent的实现如下：
function logEvent(name, params) {      
  params["event"] = name

  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push(params)
}

```