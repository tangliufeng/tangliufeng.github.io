---
layout: post
title: "Physical Layer(物理层)"
subtitle: ""
date: 2020-10-08
author: "tangliufeng"
header-img: "img/post-bg-2015.jpg"
tags: 
    - BLE
---

# 1. Physical Layer (物理层)

物理层, 控制无线电信号的收发

![](https://draapho.github.io/images/1713/ble-channel.png)

- BLE占用的频段为 2402Mhz-2480Mhz, 划分为40个Channel(频道), 每个频段2Mhz
- 3个广播频段被放在了干扰冲突最少的频段, 并被命名为 37,38,39 频道
- 其余的频段用作数据通讯, 并会采用调频技术来避免干扰和冲突.
- BLE和wifi的重合频段还是比较严重的! 和 wifi 的 1,6,11 三个频道完全重合.