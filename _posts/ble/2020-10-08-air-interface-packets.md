---
layout: post
title: "Air Interface Packets (空口包)"
subtitle: ""
date: 2020-10-08
author: "tangliufeng"
header-img: "img/post-bg-2015.jpg"
tags: 
    - BLE
---

# 1. Air Interface Packets (空口包)

虽然BLE空口包（packet，又称air interface packet）涉及BLE协议栈link layer，L2CAP，SMP和ATT等各层次，但link layer跟空口包格式关系最紧密，掌握了BLE packet的格式，就很容易理解BLE link layer协议的工作原理，因此文章取名“详解BLE空口包格式—兼BLE link layer协议解析”


![ble packet 格式](https://images4.pianshen.com/764/b8/b83d21ecebb8a4ce25c8e06bef85149c.png)


![ble packet 格式](https://upload-images.jianshu.io/upload_images/2959133-1f5bdda99b405599)



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
![图 1](/img/in-post/post-ble/post-ble/83970dce94f15cd4518d19be5169f0579ba0a4e21c9848bc29d3aa29000a06c0.png)  


### 1.2.1. Packet Header

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




### 1.2.3. Advertising Data

待编辑



## 1.3. Data Channel PDU (数据通道空口包/数据包)

待编辑