---
layout: post
title: "Link Layer(链路层)"
subtitle: ""
date: 2020-10-08
author: "tangliufeng"
header-img: "img/post-bg-2015.jpg"
tags: 
    - BLE
---


# 1. Link Layer(链路层)

链路层（Link Layer，简写 LL）：LL层是整个BLE协议栈的核心，也是BLE协议栈的难点和重点。LL层要做的事情非常多，比如具体选择哪程度 个射频通道进行通信，怎么识别空中数据包，具体在哪个时间点把数据包发送出去，怎么保证数据的完整性，ACK如何接收，如何进行重传，以及如何对链路进行管理和控制等等。LL层只负责把数据发出去或者收回来，对数据进行怎样的解析则交给上面的GAP或者ATT。

## 1.1. 状态

五种状态:

  - 就绪态(Standby) -- 睡觉省电；上电,都进入就绪态；
  - 广播态(Advertising) -- 广播模式, 周期性地发送数据, 使用的是 37-39的广播频道；一般是从机广播 ；
  - 扫描态(Scanning) --  扫描模式, 扫描/接收广播数据；一般是主机主动扫描，从机被动扫描；
  - 发起态(Initiating) --  连接发起方，扫描带有“可连接”标志的广播数据,一旦发现,则发起连接请求；只能是将要作为主机方发起连接请求；
  - 连接态(Connection)   -- 建立点对点的通讯模式, 此后就开始使用 0-36频道开始通讯了!；分为主机和从机；

注意：发起态发送连接请求后就会进入数据通道，不管和对方有没有真正建立连接，都假设已经建立，在数据通道过程中进行数据包发送，如果能应答说明连接建立，如果发送数据包应答超时，那么连接失败发起态进入就绪态


![](http://www.wowotech.net/content/uploadfile/201603/0b4e534c15ee5ffb7cb0d24c67fa1a0f20160312143237.gif)


## 1.2. 角色

设备的几种角色:

- Advertiser: 广播者, 处于 Advertising 模式即广播者
- Scanner: 扫描者, 处于 Scanning 模式即扫描者
- Initiator: 扫描者, 处于 Initiating 模式的扫描, 用于准备建立连接
- Slave: 建立通讯后 (Connection 模式), 之前的广播者就变成了Slave从机
- Master: 建立通讯后 (Connection 模式), 之前的扫描者/发起者就变成了Master主机
- 混合模式: 一个设备可以同时是 Advertiser & Scanner 或者 Advertiser & Master 等等. 但需要供应商的软件支持!


  _关于蓝牙设备的角色名称问题, 无力吐槽, 各个分层都有一套自己的术语, 看的时候注意其对应的架构层次再来理解_

