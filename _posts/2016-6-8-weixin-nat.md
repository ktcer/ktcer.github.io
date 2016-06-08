---
layout: post
title: Android微信智能心跳方案(转载)
categories: Android
description: Android微信智能心跳方案。
keywords: Android,智能心跳
---

**  方案的主要目标是，在尽量不影响用户收消息及时性的前提下，根据网络类型自适应的找出保活信令TCP连接的尽可能大的心跳间隔，从而达到减少安卓微信因心跳引起的空中信道资源消耗，减少心跳Server的负载，以及减少部分因心跳引起的耗电。

# [详情请看微信开发团队公众号](https://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207243549&idx=1&sn=4ebe4beb8123f1b5ab58810ac8bc5994&scene=1&key=dffc561732c22651a7a551503a3e34117fa2cfa4d0eabcfb2a8974dbc8bb9e038324ec7286885b7e855b9272585eee44&ascene=0&uin=MjMyNzA5NjUwMA%3D%3D&devicetype=iMac+MacBookPro11%2C1+OSX+OSX+10.10.5+build(14F27)&version=11020113&pass_ticket=ka0fho%2BQmBJa%2FIrVUZ%2B%2F5D9jGw1RgvUIpCZINFEgomTDYSrSKYrIIGZRgS%2BwFBFP)
