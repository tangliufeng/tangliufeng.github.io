---
layout: post
title: "Air Interface Packets (空口包)"
subtitle: "[Vol 6, Part B]-Link Layer Specification-2"
date: 2020-10-08
author: "tangliufeng"
header-img: "img/post-bg-2015.jpg"
tags: 
    - BLE
---

# Air Interface Packets (空口包)


虽然BLE空口包（packet，又称air interface packet）涉及BLE协议栈link layer，L2CAP，SMP和ATT等各层次，但link layer跟空口包格式关系最紧密。


![](https://images4.pianshen.com/764/b8/b83d21ecebb8a4ce25c8e06bef85149c.png)


![](https://upload-images.jianshu.io/upload_images/2959133-1f5bdda99b405599)



## Packet Format (格式)


BLE链路层（link layer）只定义了一种packet（空中包）格式，如下所示：

![](https://oscimg.oschina.net/oscnet/551aa19f3e04fa65de442df077a98440318.png)

而且PDU（protocol data unit，协议数据单元）前两个字节固定为LL header（1个字节长）和payload length（1个字节长，又称data length），即上面的Packet可以展开为：

![](https://oscimg.oschina.net/oscnet/7527a9806ecf3ac631afca26a19a699bb77.png)

### Preamble (前导帧)


preamble（前导帧）为1个字节，根据Access Address第一个Bit，有两种取值情况：0x55或者0xAA（纯PHY层行为），如下所示：

![](https://oscimg.oschina.net/oscnet/df4d79612abf0c66502fb28b7be58545905.png)

### Access Address

用来标示接收者ID或者空中包身份，如前所示，BLE只有一种packet格式，根据Access Address的不同，又区分两种Packet类型：广播包和数据包：

- 广播包Access Address固定为 0x8E89BED6
    
广播包只能在广播信道（channel）上传输，即只能在37/38/39信道上传输（注：从蓝牙5.0开始广播包可以在其它信道上传输）。广播包发送给附近所有的observer（扫描者）

- 数据包Access Address为一个32bit的随机值

由Initiator生成。数据包，其实是数据信道上的空中包的简称，数据包只在数据信道上传输，即除37/38/39之外的其余37信道（BLE总共占用40个信道）。每建立一次连接，重新生成一次Access address。数据包是给连接通信使用的，即用于master和slave之间通信的。

### CRC

![](https://oscimg.oschina.net/oscnet/b42bf3273a9ff90608284a7db8fe305529f.png)


## Advertising Channel PDU (蓝牙广播包)

蓝牙广播包，全名蓝牙广播通道（channel）空中包，即在蓝牙广播通道上传输的空中包，为两种空中包的一种，其具体格式如下所示：

![](https://oscimg.oschina.net/oscnet/8693fa6fbc0cdb602bedd08a4ae14c067fb.png)


Advertising channel PDU Header: 

![](/img/in-post/post-ble/83970dce94f15cd4518d19be5169f0579ba0a4e21c9848bc29d3aa29000a06c0.png)  


### Packet Header

- `Advertising Header` 即前述的LL header，长度为一个字节，其每bit定义如下所示：

    ![](https://oscimg.oschina.net/oscnet/c560564dc946e98e1fc7b32388e5d6bbb8a.png)

    - `PDU Type` 为3bit，具体定义如下。可以看出扫描PDU和发起连接PDU都属于广播包:


        | Status      | PDU Type     | Value           | Description                                                                                                         |
        | ----------- | ------------ | --------------- | :------------------------------------------------------------------------------------------------------------------ |
        | Advertising | 0b0000(0x00) | ADV_IND         | Connectable undirected advertising event                                                                            |
        |             | 0b0001(0x01) | ADV_DIRECT_IND  | Connectable directed advertising event                                                                              |
        |             | 0b0010(0x02) | ADV_NONCONN_IND | non-connectable undirected advertising events.                                                                      |
        |             | 0b0110(0x06) | ADV_SCAN_IND  | scannable undirected advertising event                                                                              |
        | Scanning    | 0b0011(0x03) | SCAN_REQ        | sent by the Link Layer in the Scanning State, received by a Link Layer in the Advertising State                     |
        |             | 0b0100(0x04) | SCAN_RSP        | sent by the Link Layer in the Advertising State, received by aLink Layer in the Scanning State                      |
        | Initiating  | 0b0101(0x05) | CONNECT_REQ     | This PDU is sent by the Link Layer in the Initiating State and received by the Link Layer in the Advertising State. |

  - `TxAdd/RxAdd`，各占1bit，表示随后的Device Address字段代表的蓝牙MAC地址类型，值0代表Public地址，值1代表Random地址。

- `Payload length`定义如下所示，所以广播包PDU最长37个字节:
  
  ![](https://oscimg.oschina.net/oscnet/da2535f6759d6749367a28755ce4cea3f7a.png)


### PDUs Payload



- `Device Address(Advertising Address/AdvA)`，广播包中的强制字段，俗称蓝牙MAC地址，如果是广播包，则是advertiser的MAC地址；如果是scan包或者连接请求包，则是scanner的MAC地址。蓝牙device address为6个字节，这样Advertising data最长为：37-6 = 31B，这就是广播包数据最长只能31个字节的由来。如前所述，device address分`public`和`random`两种，定义如下所示：
![](https://oscimg.oschina.net/oscnet/a15c2c6d91a08e2b3589146e05259f6eb35.png)


  - Random device address又有三种类型，定义如下所示：

    ![](https://oscimg.oschina.net/oscnet/57d854de127b33f052296208f9fd49397e9.png)


#### Advertising PDUs

- `ADV_IND`: connectable undirected advertising event
- `ADV_DIRECT_IND`: connectable directed advertising event
- `ADV_NONCONN_IND`: non-connectable undirected advertising event
- `ADV_SCAN_IND`: scannable undirected advertising event


##### ADV_IND

![图 2](/img/in-post/post-ble/778f2aa9ee96196bf24508581463b0b393fb7412aebf5f8b1f3a354232c360fa.png)  

##### ADV_DIRECT_IND

![图 3](/img/in-post/post-ble/495e60999dc2a8297627e7468d3a999ab588304d2e04059ee332685469f2576f.png)  

##### ADV_NONCONN_IND

![图 4](/img/in-post/post-ble/8c7a81c9476d4150bbae3a7ec547d0f79844497fc06edd72298b3b0f6821be9f.png)  

##### ADV_SCAN_IND

![图 5](/img/in-post/post-ble/86d44b01eac263c51a33309da88a81d600e01a2769e938355a283a5973e4aa64.png)  


#### Scanning PDUs

- `SCAN_REQ`: sent by the Link Layer in the Scanning State, received by a
Link Layer in the Advertising State
- `SCAN_RSP`: sent by the Link Layer in the Advertising State, received by a
Link Layer in the Scanning State

##### SCAN_REQ

![图 6](/img/in-post/post-ble/d8b7c8dbef78fb90cdf5462bb050af15a9643fa909faf6d79b64dfb4188cdfdf.png)  

##### SCAN_RSP

![图 7](/img/in-post/post-ble/67659096cb851b58aca5fb055184fef7c00dbaf25ed96a4eff26a147c86caa1e.png)  

#### Initiating PDUs

##### CONNECT_REQ

![图 8](/img/in-post/post-ble/d878535aa159a47b6547e0c9625aa0e8ee39142c8c96441934ac28c19d396d79.png)  

![图 9](/img/in-post/post-ble/96c41981abd121fa7f6dcec548ce6fe355683d17c0736d3a0ec0788621e75bcf.png)  

LL Data 包含10个字段:

- `AA` 4个字节。AA字段应包含链路层连接的访问地址。
- `CRCInit` 用于CRC计算的一个初始值，由Link Layer随机生成。
- `WinSize` 指示transmitWindowSize值。transmitWindowSize = WinSize * 1.25 ms。
- `WinOffset` 指示transmitWindowOffset值。transmitWindowOffset = WinOffset * 1.25 ms。
- `Interval` 指示connInterval。connInterval = Interval * 1.25 ms
- `Latency`  指示connSlaveLatency值。connSlaveLatency =Latency
- `Timeout` 指示connSupervisionTimeout。connSupervisionTimeout = Timeout * 10 ms
- `ChM` 标识当前使用和未使用的Physical Channel
- `Hop` 指示hopIncrement，它和ChM一起决定了数据传输过程中的跳频算法。
- `SCA` 用于定义最差的Master睡眠时钟精度。
  


## Data Channel PDU (数据通道空口包/数据包)

待编辑


连接态的数据包我们统称为  Data Channel PDU ，与  Advertising Channel PDU 不同，Data Channel PDU 允许数据在除了 37、38、39信道上的其他的 37 个信道上进行数据传输，根据用途，又将其分为两种：

- LL Data PDU：纯数据包
- LL Control PDU：控制包

普通数据包格式如下：

![](https://upload-images.jianshu.io/upload_images/2959133-e622712e22876880.png?imageMogr2/auto-orient/strip|imageView2/2/w/687/format/webp)

![](https://www.pianshen.com/images/529/85b5ebe4d37667ea1f9c28f3b9fb5d89.png)

Data header，即前述的LL header，在数据包中的定义如下所示：

![](https://upload-images.jianshu.io/upload_images/2959133-a63385dcf2f11835.png?imageMogr2/auto-orient/strip|imageView2/2/w/689/format/webp)

- LLID ： 用于区分这个 Connection 的包是普通的数据包（L2CAP 的起始/连续/空包），还是 Control PDU

  - 00b = Reserved 
  - 01b = LL Data PDU: Continuation fragment of an L2CAP message, or an Empty PDU.
  - 10b = LL Data PDU: Start of an L2CAP message or a complete L2CAP message with no fragmentation.
  - 11b = LL Control PDU

- NESN：下一个期望的对端包的 Sequence Number
- SN：当前包的 Sequence Number
- MD：是否有 More Data
- RFU：预留
- Length：Playload （含 MIC） 的数据长度，单位是字节
  

## 1.2. LL Control PDU

控制 PDU （也成为 LLCP）主要用于建立连接后的一些参数设置，流程交互等等：

![](https://www.pianshen.com/images/663/79b86f37d7de1e2370fc3dc73bcdbd67.png)


不同的 Opcode 代表了不同的控制包：

| Opcode           | Control PDU Name         | Description              |
| ---------------- | ------------------------ | ------------------------ |
| 0x00             | LL_CONNECTION_UPDATE_IND | 更新链接参数             |
| 0x01             | LL_CHANNEL_MAP_IND       | 更新链接的 Channel Map   |
| 0x02             | LL_TERMINATE_IND         | 断开连接请求             |
| 0x03             | LL_ENC_REQ               | 加密流程相关交互         |
| 0x04             | LL_ENC_RSP               |                          |
| 0x05             | LL_START_ENC_REQ         |                          |
| 0x06             | LL_START_ENC_RSP         |                          |
| 0x07             | LL_UNKNOWN_RSP           | 收到未知的 LLCP 后的回复 |
| 0x08             | LL_FEATURE_REQ           | 请求交换 Feature 的交互  |
| 0x09             | LL_FEATURE_RSP           |                          |
| 0x0A             | LL_PAUSE_ENC_REQ         | 重启加密流程相关交互     |
| 0x0B             | LL_PAUSE_ENC_RSP         |                          |
| 0x0C             | LL_VERSION_IND           | 交互 Version             |
| 0x0D             | LL_REJECT_IND	拒绝请求   |
| 0x0E             | LL_SLAVE_FEATURE_REQ     | Slave 请求 Feature       |
| 0x0F             | LL_CONNECTION_PARAM_REQ  | 更新链接参数             |
| 0x10             | LL_CONNECTION_PARAM_RSP  |
| 0x11             | LL_REJECT_EXT_IND        | 扩展类型的拒绝请求       |
| 0x12             | LL_PING_REQ              | 加密后的 PING 流程交互   |
| 0x13             | LL_PING_RSP              |
| 0x14             | LL_LENGTH_REQ            | 更新空口数据长度         |
| 0x15             | LL_LENGTH_RSP            |
| 0x16             | LL_PHY_REQ               | PHY更新相关交互          |
| 0x17             | LL_PHY_RSP               |
| 0x18             | LL_PHY_UPDATE_IND        |
| 0x19             | LL_MIN_USED_CHANNELS_IND | Channels 相关的配置      |
| All other values | Reserved for Future Use  | 预留                     |


### LL Data PDU

### LL Control PDU

