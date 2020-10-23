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

# 1. Air Interface Packets (空口包)


虽然BLE空口包（packet，又称air interface packet）涉及BLE协议栈link layer，L2CAP，SMP和ATT等各层次，但link layer跟空口包格式关系最紧密。


![](https://images4.pianshen.com/764/b8/b83d21ecebb8a4ce25c8e06bef85149c.png)


![](https://upload-images.jianshu.io/upload_images/2959133-1f5bdda99b405599)



## 1.1. Packet Format (格式)


BLE链路层（link layer）只定义了一种packet（空中包）格式，如下所示：

![](https://oscimg.oschina.net/oscnet/551aa19f3e04fa65de442df077a98440318.png)

而且PDU（protocol data unit，协议数据单元）前两个字节固定为LL header（1个字节长）和payload length（1个字节长，又称data length），即上面的Packet可以展开为：

![](https://oscimg.oschina.net/oscnet/7527a9806ecf3ac631afca26a19a699bb77.png)

### 1.1.1. Preamble (前导帧)


preamble（前导帧）为1个字节，根据Access Address第一个Bit，有两种取值情况：0x55或者0xAA（纯PHY层行为），如下所示：

![](https://oscimg.oschina.net/oscnet/df4d79612abf0c66502fb28b7be58545905.png)

### 1.1.2. Access Address

用来标示接收者ID或者空中包身份，如前所示，BLE只有一种packet格式，根据Access Address的不同，又区分两种Packet类型：广播包和数据包：

- 广播包Access Address固定为 0x8E89BED6
    
广播包只能在广播信道（channel）上传输，即只能在37/38/39信道上传输（注：从蓝牙5.0开始广播包可以在其它信道上传输）。广播包发送给附近所有的observer（扫描者）

- 数据包Access Address为一个32bit的随机值

由Initiator生成。数据包，其实是数据信道上的空中包的简称，数据包只在数据信道上传输，即除37/38/39之外的其余37信道（BLE总共占用40个信道）。每建立一次连接，重新生成一次Access address。数据包是给连接通信使用的，即用于master和slave之间通信的。

### 1.1.3. CRC

![](https://oscimg.oschina.net/oscnet/b42bf3273a9ff90608284a7db8fe305529f.png)


## 1.2. Advertising Channel PDU (蓝牙广播包)

蓝牙广播包，全名蓝牙广播通道（channel）空中包，即在蓝牙广播通道上传输的空中包，为两种空中包的一种，其具体格式如下所示：

![](https://oscimg.oschina.net/oscnet/8693fa6fbc0cdb602bedd08a4ae14c067fb.png)


Advertising channel PDU Header: 

![](/img/in-post/post-ble/83970dce94f15cd4518d19be5169f0579ba0a4e21c9848bc29d3aa29000a06c0.png)  


### 1.2.1. Packet Header

- `Advertising Header` 即前述的LL header，长度为一个字节，其每bit定义如下所示：

    ![](https://oscimg.oschina.net/oscnet/c560564dc946e98e1fc7b32388e5d6bbb8a.png)

    - `PDU Type` 为3bit，具体定义如下。可以看出扫描PDU和发起连接PDU都属于广播包:


        | Status      | PDU Type     | Value           | Description                                                                                                         |
        | ----------- | ------------ | --------------- | :------------------------------------------------------------------------------------------------------------------ |
        | Advertising | 0b0000(0x00) | ADV_IND         | Connectable undirected advertising event                                                                            |
        |             | 0b0001(0x01) | ADV_DIRECT_IND  | Connectable directed advertising event                                                                              |
        |             | 0b0010(0x02) | ADV_NONCONN_IND | non-connectable undirected advertising events.                                                                      |
        |             | 0b0110(0x06) | ADV_SCAN_IND    | scannable undirected advertising event                                                                              |
        | Scanning    | 0b0011(0x03) | SCAN_REQ        | sent by the Link Layer in the Scanning State, received by a Link Layer in the Advertising State                     |
        |             | 0b0100(0x04) | SCAN_RSP        | sent by the Link Layer in the Advertising State, received by aLink Layer in the Scanning State                      |
        | Initiating  | 0b0101(0x05) | CONNECT_REQ     | This PDU is sent by the Link Layer in the Initiating State and received by the Link Layer in the Advertising State. |

  - `TxAdd/RxAdd`，各占1bit，表示随后的Device Address字段代表的蓝牙MAC地址类型，值0代表Public地址，值1代表Random地址。

- `Payload length`定义如下所示，所以广播包PDU最长37个字节:
  
  ![](https://oscimg.oschina.net/oscnet/da2535f6759d6749367a28755ce4cea3f7a.png)


### 1.2.2. PDUs Payload



- `Device Address(Advertising Address/AdvA)`，广播包中的强制字段，俗称蓝牙MAC地址，如果是广播包，则是advertiser的MAC地址；如果是scan包或者连接请求包，则是scanner的MAC地址。蓝牙device address为6个字节，这样Advertising data最长为：37-6 = 31B，这就是广播包数据最长只能31个字节的由来。如前所述，device address分`public`和`random`两种，定义如下所示：
![](https://oscimg.oschina.net/oscnet/a15c2c6d91a08e2b3589146e05259f6eb35.png)


  - Random device address又有三种类型，定义如下所示：

    ![](https://oscimg.oschina.net/oscnet/57d854de127b33f052296208f9fd49397e9.png)


#### 1.2.2.1. Advertising PDUs

- `ADV_IND`: connectable undirected advertising event
- `ADV_DIRECT_IND`: connectable directed advertising event
- `ADV_NONCONN_IND`: non-connectable undirected advertising event
- `ADV_SCAN_IND`: scannable undirected advertising event


##### 1.2.2.1.1. ADV_IND

![图 2](/img/in-post/post-ble/778f2aa9ee96196bf24508581463b0b393fb7412aebf5f8b1f3a354232c360fa.png)  

##### 1.2.2.1.2. ADV_DIRECT_IND

![图 3](/img/in-post/post-ble/495e60999dc2a8297627e7468d3a999ab588304d2e04059ee332685469f2576f.png)  

##### 1.2.2.1.3. ADV_NONCONN_IND

![图 4](/img/in-post/post-ble/8c7a81c9476d4150bbae3a7ec547d0f79844497fc06edd72298b3b0f6821be9f.png)  

##### 1.2.2.1.4. ADV_SCAN_IND

![图 5](/img/in-post/post-ble/86d44b01eac263c51a33309da88a81d600e01a2769e938355a283a5973e4aa64.png)  


#### 1.2.2.2. Scanning PDUs

- `SCAN_REQ`: sent by the Link Layer in the Scanning State, received by a
Link Layer in the Advertising State
- `SCAN_RSP`: sent by the Link Layer in the Advertising State, received by a
Link Layer in the Scanning State

##### 1.2.2.2.1. SCAN_REQ

![图 6](/img/in-post/post-ble/d8b7c8dbef78fb90cdf5462bb050af15a9643fa909faf6d79b64dfb4188cdfdf.png)  

##### 1.2.2.2.2. SCAN_RSP

![图 7](/img/in-post/post-ble/67659096cb851b58aca5fb055184fef7c00dbaf25ed96a4eff26a147c86caa1e.png)  

#### 1.2.2.3. Initiating PDUs

##### 1.2.2.3.1. CONNECT_REQ

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
  


## 1.3. Data Channel PDU (数据通道空口包)


连接态的数据包我们统称为  Data Channel PDU ，与  Advertising Channel PDU 不同，Data Channel PDU 允许数据在除了 37、38、39信道上的其他的 37 个信道上进行数据传输，根据用途，又将其分为两种：

- LL Data PDU：纯数据包
- LL Control PDU：控制包

普通数据包格式如下：

![](https://upload-images.jianshu.io/upload_images/2959133-e622712e22876880.png)

![](https://www.pianshen.com/images/529/85b5ebe4d37667ea1f9c28f3b9fb5d89.png)

Data header，即前述的LL header，在数据包中的定义如下所示：

![](https://upload-images.jianshu.io/upload_images/2959133-a63385dcf2f11835.png)

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
  

### 1.3.1. LL Data PDU

数据包 是 空口包的一种,用于发送L2CAP数据。`LLID` 字段在Header应该设置为01b或10b。

当数据包的 `LLID` 字段在Header为 01b,且`Length`字段为00000000b,就称为空包。主设备在Link Layer(链路层)可发送空包给从设备,且允许从设备应答任意的空口包(Data Channel PDU),包含空包。

当数据包的 `LLID` 字段在Header为 01b,不允许`Length`字段设为00000000b。


### 1.3.2. LL Control PDU

![](https://www.pianshen.com/images/663/79b86f37d7de1e2370fc3dc73bcdbd67.png)

控制包（也是空口包的一种）主要用于建立连接后的一些参数设置，流程交互等等：

控制包的负载(Payload),包含Opcode和CtrData如下图:

![](/img/in-post/post-ble/1cf0f0105f8ecc8369a32148fc2a538bf1a74787960fdf777e8b4914ce4a0c2f.png)  


控制包不应该有`Length`字段为00000000b。所有控制包依据不同的Opcode,都有固定的`Length`。




不同的 Opcode 代表了不同的控制包：

| Opcode           | Control PDU Name         | Description              |
| ---------------- | ------------------------ | ------------------------ |
| 0x00             | LL_CONNECTION_UPDATE_IND | 更新链接参数             |
| 0x01             | LL_CHANNEL_MAP_IND       | 更新链接的 Channel Map   |
| 0x02             | LL_TERMINATE_IND         | 断开连接请求             |
| 0x03             | LL_ENC_REQ               | 加密流程相关请求         |
| 0x04             | LL_ENC_RSP               | 加密流程相关应答         |
| 0x05             | LL_START_ENC_REQ         |                          |
| 0x06             | LL_START_ENC_RSP         |                          |
| 0x07             | LL_UNKNOWN_RSP           | 收到未知的 LLCP 后的应答 |
| 0x08             | LL_FEATURE_REQ           | 请求交换 Feature 请求    |
| 0x09             | LL_FEATURE_RSP           | 请求交换 Feature 应答    |
| 0x0A             | LL_PAUSE_ENC_REQ         | 重启加密流程相关请求     |
| 0x0B             | LL_PAUSE_ENC_RSP         | 重启加密流程相关应答     |
| 0x0C             | LL_VERSION_IND           | 交互 Version             |
| 0x0D             | LL_REJECT_IND            | 拒绝请求                 |
| 0x0E             | LL_SLAVE_FEATURE_REQ     | Slave 请求 Feature       |
| 0x0F             | LL_CONNECTION_PARAM_REQ  | 更新链接参数请求         |
| 0x10             | LL_CONNECTION_PARAM_RSP  | 更新链接参数应答         |
| 0x11             | LL_REJECT_EXT_IND        | 扩展类型的拒绝请求       |
| 0x12             | LL_PING_REQ              | 加密后的 PING 流程请求   |
| 0x13             | LL_PING_RSP              | 加密后的 PING 流程应答   |
| 0x14             | LL_LENGTH_REQ            | 更新空口数据长度请求     |
| 0x15             | LL_LENGTH_RSP            | 更新空口数据长度应答     |
| 0x16             | LL_PHY_REQ               | PHY更新请求              |
| 0x17             | LL_PHY_RSP               | PHY更新应答              |
| 0x18             | LL_PHY_UPDATE_IND        |
| 0x19             | LL_MIN_USED_CHANNELS_IND | Channels 相关的配置      |
| 0x1A             | LL_CTE_REQ               |                          |
| 0x1B             | LL_CTE_RSP               |                          |
| 0x1C             | LL_PERIODIC_SYNC_IND     |                          |
| 0x1D             | LL_CLOCK_ACCURACY_REQ    |                          |
| 0x1E             | LL_CLOCK_ACCURACY_RSP    |                          |
| 0x1F             | LL_CIS_REQ               |                          |
| 0x20             | LL_CIS_RSP               |                          |
| 0x21             | LL_CIS_IND               |                          |
| 0x22             | LL_CIS_TERMINATE_IND     |                          |
| 0x23             | LL_POWER_CONTROL_REQ     |                          |
| 0x24             | LL_POWER_CONTROL_RSP     |                          |
| 0x25             | LL_POWER_CHANGE_IND      |                          |
| All other values | Reserved for Future Use  | 预留                     |

CtrData 根据 Opcode类型不同,定义也不一样。 

如果接收到Opcode为不支持或保留的控制包,链路层(Link Layer)应以`LL_UNKNOWN_RSP`的控制包应答。LL_UNKNOWN_RSP控制包中的`UnknownType`字段应设置为接收到的控制包的Opcode。

如果接收到错误的长度或无效CtrData字段,链路层可以继续有关链路过程和实现的解释数据(例如,如果PDU太长可以忽略额外的数据;如果字段超出了范围，则可以使用最近被允许的值)。如果不继续链路过程，则响应
`LL_UNKNOWN_RSP` 或者如果允许继续相关链路过程,则应答`LL_REJECT_IND` 、`LL_REJECT_EXT_IND` 。`LL_UNKNOWN_RSP`控制包的`UnknownType`字段或者`LL_REJECT_EXT_IND`控制包的`RejectOpcode`字段应该设置为接收的控制包的`Opcode`值。

#### 1.3.2.1. LL_CONNECTION_UPDATE_IND

CtrData字段格式如下图:

![](/img/in-post/post-ble/84ebc26aae41b11fdd17b7cb85a1d5cc8fa38a67566e208c5b1030856acd460b.png)  
The Instant field shall be set to indicate the instant described in Section

- WinSize: 用于指示transmitWindowSize的值。transmitWindowSize = WinSize * 1.25 ms
- WinOffset: 用于指示transmitWindowOffset的值。transmitWindowOffset = WinOffset * 1.25 ms
- Interval: 用于指示connInterval的值。connInterval = Interval * 1.25 ms
- Latency: 用于指示connSlaveLatency的值。connSlaveLatency =Latency
- Timeout: 用于指示connSupervisionTimeout的值。connSupervisionTimeout = Timeout * 10 ms
- Instant: 范围0~65536
  
#### 1.3.2.2. LL_CHANNEL_MAP_IND

CtrData字段格式如下图:
![](/img/in-post/post-ble/c54abebf2f473f114f31951a3606dc6334277af9e97c06763d010d2686d8d4b9.png)  

- ChM: 已使用、未使用的数据通道映射。每个通道用1 bit表示。
- Instant: 
  
#### 1.3.2.3. LL_TERMINATE_IND

CtrData字段格式如下图:
![](/img/in-post/post-ble/541430a7326ebe47984d6fd6f1176120a93e19b0a7659d948a751d10312c782b.png) 

- ErrorCode: 通知远程设备为什么连接被终止。参见[Vol 1] Part F, Controller Error Codes。


#### 1.3.2.4. LL_ENC_REQ

#### 1.3.2.5. LL_ENC_RSP

#### 1.3.2.6. LL_START_ENC_REQ

#### 1.3.2.7. LL_START_ENC_RSP

#### 1.3.2.8. LL_UNKNOWN_RSP

#### 1.3.2.9. LL_FEATURE_REQ


#### 1.3.2.10. LL_FEATURE_RSP

#### 1.3.2.11. LL_PAUSE_ENC_REQ

#### 1.3.2.12. LL_PAUSE_ENC_RSP
#### 1.3.2.13. LL_VERSION_IND
#### 1.3.2.14. LL_REJECT_IND

#### 1.3.2.15. LL_SLAVE_FEATURE_REQ

#### 1.3.2.16. LL_CONNECTION_PARAM_REQ

#### 1.3.2.17. LL_CONNECTION_PARAM_RSP
#### 1.3.2.18. LL_REJECT_EXT_IND

#### 1.3.2.19. LL_PING_REQ

#### 1.3.2.20. LL_PING_RSP

#### 1.3.2.21. LL_LENGTH_REQ and LL_LENGTH_RSP
#### 1.3.2.22. LL_PHY_REQ and LL_PHY_RSP


