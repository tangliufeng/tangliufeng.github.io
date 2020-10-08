---
layout: post
title: "Air Interface Protocol(空口协议)"
subtitle: ""
date: 2020-10-08
author: "tangliufeng"
header-img: "img/post-bg-2015.jpg"
tags: 
    - BLE
---


#  1. Air Interface Protocol

## 1.1. Link Layer Device Filtering (设备过滤机制)

### 1.1.1. White List (白名单)

每一个BLE的Controller，可以保存一个设备列表，通过该列表，可以实现设备过滤的功能。这个列表就称作白名单（White List），保存了一些BLE设备地址。

白名单的大小由Controller自行觉得，并在reset的时候为空，后续可以由Host通过HCI接口配置。基于白名单，Link Layer可实现多种设备过滤的策略，包括：

- Advertising Filter Policy
- Scanner Filter Policy
- Initiator Filter Policy

### 1.1.2. Advertising Filter Policy

Advertising Filter Policy定义了Advertiser（处于Advertising状态）的Link Layer怎么处理SCAN_REQ（扫描请求）和CONNECT_REQ（连接请求），包括如下策略（Host可以根据实际情况配置，同一时刻只能配置一种）：


- Link Layer只接受位于白名单中的设备的扫描和连接请求（最严格）；
- Link Layer可以接受任何设备的扫描和连接请求（最不严格，Controller reset后的默认状态）；
- Link Layer可以接受任何设备的扫描请求，但只接受位于白名单中的设备的连接请求；
- Link Layer可以接受任何设备的连接请求，但只接受位于白名单中的设备的扫描请求。

### 1.1.3. Scanner Filter Policy

Scanner Filter Policy定一个Scanner（处于Scanning状态）的Link Layer怎么处理广播数据，包括如下策略：

- Link Layer只处理位于白名单中的设备的广播数据，并且忽略没有包括自身地址的connectable Directed advertising packet；
- Link Layer处理所有设备的广播数据，并且忽略没有包括自身地址的connectable Directed advertising packet（Controller reset后的默认状态）。

另外，如果设备支持“Extended Scanner Filter”策略，则可以同时支持如下的策略：

- Link Layer只处理位于白名单中的设备的广播数据，并且不能忽略InitA地址为“resolvable private address”的connectable Directed advertising packet；
- Link Layer处理所有设备的广播数据，并且不能忽略InitA地址为“resolvable private address”的connectable Directed advertising packet。

### 1.1.4. Initiator Filter Policy

Initiator Filter Policy定一个Initiator （处于Initiating状态）的Link Layer怎么处理可连接的广播数据，包括如下策略：

- Link Layer只处理位于白名单中的设备发送的可连接的广播包，并在收到的时候发起连接请求；
- 忽略白名单，Link  Layer处理由Host指定的设备所发送的可连接的广播包，并在收到的时候发起连接请求。

## 1.2. Non-Connected States(非连接状态)


### 1.2.1. Standby State

### 1.2.2. Advertising State(广播态)


`Advertising Event`是在所有被使用的物理Channel上，发送的Advertising PDU的组合。

#### 1.2.2.1. Advertising Event

    蓝牙 BLE 业务都是周期性事件，Advertising 也不例外，一个 Advertising Event 的被定义成为，一次 ADV 业务，对于 Legacy 来说，就是在 37、38、39 上个各发出一个 ADV 包（有交互，就交互），对于 Extend ADV 来说，是完成 37、38、39 上个各发出一个 EXT ADV 包后，在第二通道上发出其对应的包的全过程（有交互，就交互）。


广播事件分类:

- 非定向可连接事件(ADV_IND)
- 定向可连接事件(ADV_DIRECT_IND)
- 非定向扫描事件(ADV_SCAN_IND)
- 非定向不可连接事件(ADV_NONCONN_IND)



#### 1.2.2.2. Advertising Channel Selection 

BLE可以使用40个Physical Channel中的3个作为广播通信的物理信道，综合各种因素（抗干扰等），最终选取了如下三个：

| RF Channel | RF Center Frequency | Advertising Channel Index |
|------------|---------------------|---------------------------|
| 0          | 2402MHz             | 37                        |
| 12         | 2426MHz             | 38                        |
| 39         | 2480MHz             | 39                        |

与此同时，Link Layer允许Host在这这三个物理信道中，任意选取一个或者多个，用于广播。Link Layer将相同的广播数据，在每一个被中的Channel中，发送一次。


#### 1.2.2.3. Advertising Interval


![](.//img/in-post/post-ble/4.1.png)


设备每次广播时，会在3个广播信道上发送相同的报文。这些报文被称为一个广播事件。除了定向报文以外，其他广播事件均可以选择“20ms ~ 10.24s”不等的间隔。通常，一个广播中的设备会每一秒广播一次。两个相邻广播事件之间的时间称为广播间隔。


但是，设备周期性的发送广播会有一个问题：由于设备间的时钟会不同程度的漂移，两个设备可能在很长一段时间同时广播而造成干扰。为防止这一情况的发生，除定向广播之外的其他广播类型，发送时间均会被扰动。实现该扰动的方式为，在上一次广播事件后加入“0 ~ 10ms”的随机延时。这意味着，即使两个设备广播间隔相同，并在相同信道及时间点上发送造成了冲突，但它们发送下一个广播事件时也会有很大可能不再冲突。

所以，两个相邻的广播事件的之间的时间间隔（T_advEvent）为：

    T_AdvEvent = advInterval + advDelay

其中，advInterval 必须是“0.625ms”的整数倍，范围是“20ms ~ 10.24s”之间。对于可扫描非定向广播和不可连接非定向广播这两种广播类型，该值最好不小于100ms，即（160个0.625ms）。advDelay是Link Layer（链接层）分配的一个伪随机数，它的范围为“0 ~ 10ms”。


#### 1.2.2.4. Connectable Undirected Event Type (可连接非定向广播)

这是一种用途最广的广播类型，包括广播数据和扫描响应数据，它表示当前设备可以接受其他任何设备的连接请求。进行通用广播的设备能够被扫描设备扫描到，或者在接收到连接请求时作为从设备进入一个连接。通用广播可以在没有连接的情况下发出，换句话说，没有主从设备之分。

鉴于此种广播类型用的最多，下面我们来讨论一下此类型下广播事件中广播包的发送情况，另外要注意在一个广播事件中，前一个“ADV_IND PDUs”的开始到相邻的下一个“ADV_IND PDUs”的开始处的时间要小于等于10ms ：

第一种情况：仅仅有广播 PDUs 。截图显示如下：

![](.//img/in-post/post-ble/4.2.png)

第二种情况：在广播事件的中间有“SCAN_REQ”和“SCAN_RSP PDUs”。截图显示如下：

![](/images/4.3.png)

第三种情况：在广播事件的结尾有“SCAN_REQ”和“SCAN_RSP PDUs”。截图显示如下：

![](.//img/in-post/post-ble/4.4.png)

第四种情况：在广播事件的中间接收到“CONNECT_REQ PDU”的情况。截图显示如下:

![](.//img/in-post/post-ble/4.5.png)

#### 1.2.2.5. Connectable Directed Event Type

- Low Duty Cycle 
- High Duty Cycle

定向广播类型是为了尽可能快的建立连接。这种报文包含两个地址：广播者的地址和发起者的地址。发起者收到发给自己的定向广播报文之后，可以立即发送连接请求作为回应。 

定向广播类型有特殊的时序要求。完整的广播事件必须每3.75ms重复一次。这一要求使得扫描设备只需扫描3.75ms便可以收到定向广播设备的消息。 

当然，如此快的发送会让报文充斥着广播信道，进而导致该区域内的其他设备无法进行广播。因此，定向广播不可以持续1.28s以上的时间。如果主机没有主动要求停止，或者连接没有建立，控制器都会自动停止广播。一旦到了1.28s，主机便只能使用间隔长得多的可连接非定向广播让其他设备来连接。

当使用定向广播时，设备不能被主动扫描。此外，定向广播报文的净荷中也不能带有其他附加数据。该净荷只能包含两个必须的地址。

**Low Duty Cycle**

![](.//img/in-post/post-ble/4.6.png)
![](.//img/in-post/post-ble/4.7.png)

**High Duty Cycle**

![](.//img/in-post/post-ble/4.8.png)


#### 1.2.2.6. Scannable Undirected Event Type

又称可发现广播，这种广播不能用于发起连接，但允许其他设备扫描该广播设备。这意味着该设备可以被发现，既可以发送广播数据，也可以响应扫描发送扫描回应数据，但不能建立连接。这是一种适用于广播数据的广播形式，动态数据可以包含于广播数据之中，而静态数据可以包含于扫描响应数据之中。

这个广播其实是一个非定向可发现的广播，它和通用广播的时间
控制是一样的，应答也是 SCAN_REQ PDU 和 SCAN_RSP PDU，这个广
播和通用广播的区别是，它不能建立连接， 只能处于广播态或者就绪态。这是一种适用于广播数据的广播形式，动态数据可以包含于广播数据中， 而静态数据可以包含于扫描响应数据之中。 当广播者接收到一个来自扫描者的SCAN_REQ PDU 包，包里含有广播者的设备地址，并通过了广播过滤策略时，广播者回复SCAN_RSP PDU 。如果没有通过广播过滤策略，广播者会发送下一广播通道的ADV_IND PDU 或关闭广播。

![](.//img/in-post/post-ble/4.9.png)
![](.//img/in-post/post-ble/4.10.png)
![](.//img/in-post/post-ble/4.11.png)



#### 1.2.2.7. Non-connectable Undirected Event Type

仅仅发送广播数据，而不想被扫描或者连接。这也是唯一可用于只有发射机而没有接收机设备的广播类型。不可连接广播设备不会进入连接态，因此，它只能根据主机的要求在广播态和就绪态之间切换。

![](.//img/in-post/post-ble/4.12.pngg)


### 1.2.3. Scanning State (扫描态)

主机端的扫描行为和连接请求，也使用广播信道。所谓扫描，指无线信号接收机处于工作状态，所有掠过该接收机的有效信号，都会被捕获和分析。如果是可识别的BLE广播信号，则会提取信号中的有效内容，以“发现”设备。
扫描分为两类：

- 主动扫描（Active Scan）
- 被动扫描（Passive Scan）

主动扫描是指接收了广播信号后，主机端发出扫描请求包（Scan Req Data），从机收到后进一步返回扫描响应包（Scan Response Data），如下图：

![](http://bbs.wireless-tech.cn/assets/files/2019-07-11/1562851877-74950-t01d7b6306910e2cd97.png)

在主动扫描模式下，由于存在扫描响应包，从机可以在建立连接之前，提供更多的信息给主机端。扫描响应数据是在收到扫描请求后立即发出，应用程序无法掌控这一过程，因此扫描响应数据中适合存放一些静态数据。

被动扫描则不发扫描请求包。这种扫描模式下，BLE设备只听不问，也就是说，只接收ADV_DIRECT_IND、ADV_IND、ADV_SCAN_IND、ADV_NONCONN_IND等类型的PDU，并不发送SCAN_REQ。


每次扫描都会持续一段时间，这段时间称为“扫描窗口”(Scan Window)，相邻扫描窗口之间的时间为“扫描间隔”(Scan Interval)。

![](https://draapho.github.io/images/1713/ble-scan.PNG)

_BLE协议规定，scanWindow和scanInterval最大不能超过10.24s，并且scanWindow不能大于scanInterval。_


### 1.2.4. Initiating State (发起态)

Initiating状态和Scanning状态类似，只不过它的关注点不一样：它不关心广播数据，只关心ADV_DIRECT_IND和ADV_IND两类消息，并在符合条件的时候，发出CONNECT_REQ，请求建立连接。

_主机扫描到有效广播信号，即发现了周边的有效设备，停止扫描退出Scanning状态。此时进入Standby状态。如果需要建立连接，则发出连接请求（CONNECT_REQ），状态机进入Initiating状态，注意现在不是Connection状态！连接请求发送完毕，主机端就进入Connection状态。从机端一旦收到连接请求，也进入Connection状态。也就是说，Initiating状态就是发送连接请求的那一瞬。（BLE Spec在这一块讲述的不清楚，《权威指南》上压根没讲。 -_-!）_

### 1.2.5. Connection State (连接态)



#### 1.2.5.1. Connection Events

![](https://draapho.github.io/images/1713/ble-connect.png)

在CONNECT_REQ之后会延时1.25ms，然后在时间t（`transmitWindowOffset` <= t <= `transmitWindowOffset` + `transmitWindowSize`）内，主机端（Master）发出一个数据包，等待T_IFS（Inter Frame Space：帧间隔），如果从机（Slave）返回响应，则完成第一个连接事件（Connection Event）。


![](http://bbs.wireless-tech.cn/assets/files/2019-07-11/1562853510-227285-stack-ll-connection-setup-master.png)

两个参数：`transmitWindowOffset` 和 `transmitWindowSize`是为了给从机做准备工作留的时间，也用来进行两个设备之间通信行为的时间同步（俗称：对表）。图中有个地方值得注意，主机发送连接请求后，即刻进入连接状态。从机并不回复一个响应信息。发送连接请求之前，都是使用广播信道，之后的数据都走数据信道。


与广播类似，连接状态下的数据传输行为也有”连接事件“和”连接间隔“的概念。

![](http://bbs.wireless-tech.cn/assets/files/2019-07-11/1562854025-88847-1366713-20180430002837207-145463955.png)

从上图可知，一个`connection event`可能发多个包。每个连接事件内，都是主机发出连接请求，（又是连接请求！）从机进行相应。BLE设备之间的数据通信，比如Write/Notify，用户数据就包含在在这些连接请求和响应数据包中。
如果用户数据为空，也依然需要进行这样的“握手”行为，以维持设备之间的连接性。

假如从机每个小时只Notify一次数据，那么频繁的握手必将消耗大量的能量，这对从机而言不友好。BLE协议提供了延迟响应（`Latency`）这个参数，比如`Latency` = 10，即主机发出10次握手包，从机收都可以置之不理，主机也表示淡定。但是从第11个包开始，从机需要回复一个握手包，提示自己仍然”活着“。如果从第11个包开始仍然没有收到从机端的握手包，主机就开始焦急的等待从机的响应，后续的每一次握手会期望得到从机的回复。


通常情况下，从机端会很识趣的进行回复。但假如从机端真的断电了，失去了回复的能力，主机也不会无休止等待下去。BLE协议提供了监控超时（`Supervision Timeout`）参数，当主机距离上次收到从机握手包开始，等待超过了这个超时参数的时间长度，主机就认为从机”挂了“，直接终止物理连接。

##### 1.2.5.1.1. Connection Parameters


######  1.2.5.1.1.1. Connection Interval

指在一个Connection Events的开始到下一个Connection Events 的开始的时间间隔。单位是1.25ms，最小是7.5ms，最大是4.0s。

###### 1.2.5.1.1.2. Slave Latency

允许Slave设备在没有数据要发的情况下，跳过一定数目的Connection Events
在这些Connection Events中不必回复master的包，这样就能更加省电。

![](http://bbs.wireless-tech.cn/assets/files/2019-07-11/1562854381-265670-v-at-tlwsqpj2uyw979x.png)


###### 1.2.5.1.1.3. Suppervision Timeout



这个参数设定了一个超时时间，如果BLE在这个时间内没有发生通讯的话，就会自动断开。单位是10ms，范围是100ms ~ 32s。


必须满足如下公式：

    Supervision Timeout > （1 +　slaveLatency）* （connectionInterval）

  _slaveLatency不能算在 supervision Timeout的时间内，否则连接就会不正常断开_



