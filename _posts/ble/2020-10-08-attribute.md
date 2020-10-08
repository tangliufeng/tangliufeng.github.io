---
layout: post
title: "Attribute protocol(属性协议层)"
subtitle: ""
date: 2020-10-08
author: "tangliufeng"
header-img: "img/post-bg-2015.jpg"
tags: 
    - BLE
---


# 1. Attribute protocol(属性协议层)

属性协议层（Attribute protocol，简写 ATT）：简单来说，ATT层用来定义用户命令及命令操作的数据，比如读取某个数据或者写某个数据。BLE协议栈中，开发者接触最多的就是ATT。BLE引入了attribute概念，用来描述一条一条的数据。Attribute除了定义数据，同时定义该数据可以使用的ATT命令，因此这一层被称为ATT层。

属性数据结构:

![](https://pic2.zhimg.com/80/v2-e1880a763a065c14b10dfb2845fcb107_720w.jpg)

- Attribute handle: 

  Attribute句柄。Client要访问Server的Attribute，都是通过这个句柄来访问的，也就是说ATT PDU一般都包含handle的值。用户在软件代码添加characteristic的时候，系统会自动按顺序地为相关attribute生成句柄。

- Attribute type:

  Attribute类型。在BLE中我们使用UUID来定义数据的类型，UUID是128 bit的，所以我们有足够的UUID来表达万事万物。其中有一个UUID非常特殊，它被蓝牙联盟采用为官方UUID，这个UUID如下所示：0000xxxx-0000-1000-8000-00805F9B34FB

  由于这个UUID众所周知，蓝牙联盟将自己定义的attribute或者数据只用16bit UUID来表示，比如0x1234，其实它也是128bit，完整表示为：
  00001234-0000-1000-8000-00805F9B34FB = 16 bit UUID 0x1234

  Attribute type一般是由service和characteristic规格来定义，站在蓝牙协议栈角度来看，ATT层定义了一个通信的基本框架，数据的基本结构，以及通信的指令，而GATT层就是前文所述的service和characteristic，GATT层用来赋予每个数据一个具体的内涵，让数据变得有结构和意义。换句话说，没有GATT层，低功耗蓝牙也可以通信起来，但会产生兼容性问题以及通信的低效率。

  BLE的属性类型是有限的，有四个大类：

  - Primary Service（首要服务项）
  - Secondary Service（次要服务项）
  - Include（包含服务项）
  - Characteristic（特征值）

  这些属性类型分别对应了指定的UUID，BLE对这些UUID与属性类型的映射关系做了规定：

  - 0x1800~0x26FF 用于服务类UUID    （包含在首要服务的值里面，通过得到首要服务的值，就知道服务类型，例如心率值服务0X180D)
  - 0x2700~0x27FF 用于标识计量单位  
  - 0x2800~0x28FF 用于区分属性类型    (一般是服务类型声明，例如首要服务0x2800,特征声明0x2803)
  - 0x2900~0x29FF 用于特性描述
  - 0x2A00~0x7FFF 用于区分特性类型

- Attribute value

  就是数据真正的值，0到512字节长。

- Attribute permissions
  
  Attribute的权限属性，权限属性不会直接在空中包中体现，而是隐含在ATT命令的操作结果中。目前主要有如下四种权限属性
  
  - 访问权限（Access Permission）- 只读、只写、读写
  - 加密权限（Encryption Permission） – 加密、不加密
  - 认证权限（Authentication Permission） – 需要认证、无需认证
  - 授权权限（Authorization Permission） – 需要授权、无需授权

  访问（Access）权限好理解，如果是只读权限，就不能对其写数据，其他类似。
  加密（Encryption）权限也好理解，就是对数据进行加密。
  认证（Authentication）是指相互确认对方身份。完成认证流程的两个设备，双方建立信任关系，二者之间的通信通道即可以认为是安全的。BLE中，“认证”过程就是配对。
  授权（Authorization）是指对授信设备开放权利。

  认证和授权功能容易混淆，其英文拼写也很相似。从上面的概念上看，授权要求设备必须是可信任的，因此授权的管控等级要高于认证——认证的设备未必被授权，授权的设备一定是认证的。理解二者关系，需要引入一个概念：Trusted Device（可信任设备）一个没有经过认证的设备，被称为Unknown Device（未知设备）；经过了认证该设备会在绑定信息中被标记为Untrusted，被称为Untrusted Device（不可信设备）；经过了认证，并且在绑定信息中被标记为Trusted的设备被称为Trusted Device（可信设备）。

  授权要求设备为Trusted Device（可信任设备）。在实际使用中，经过配对以后设备即为Untrusted Device——认证，在代码中调用API可以设置设备为Trusted Device——授权。

## 1.1. ATT PDU

- Attribute Opcode

| Attribute PDU               | Attribute Opcode | Parameters                                                     |
| --------------------------- | ---------------- | -------------------------------------------------------------- |
| Error Response              | 0x01             | Request Opcode in Error, Attribute Handle In Error, Error Code |
| Exchange MTU Request        | 0x02             | Client Rx MTU                                                  |
| Exchange MTU Response       | 0x03             | Server Rx MTU                                                  |
| Find Information Request    | 0x04             | Starting Handle, Ending Handle, UUID                           |
| Find Information Response   | 0x05             | Format, Information Data                                       |
| Find By Type Value Request  | 0x06             | Starting Handle,Ending Handle,Attribute Type,Attribute Value   |
| Find By Type Value Response | 0x07             | Handles Information List                                       |
| Read By Type Request        | 0x08             | Starting Handle, Ending Handle, UUID                           |
| Read By Type Response       | 0x09             | Length, Attribute Data List                                    |
| Read Request                | 0x0A             | Attribute Handle                                               |
| Read Response               | 0x0B             | Attribute Value                                                |
| Read Blob Request           | 0x0C             | Attribute Handle,Value Offset                                  |
| Read Blob Response          | 0x0D             | Part Attribute Value                                           |
| Read Multiple Request       | 0x0E             | Handle Set                                                     |
| Read Multiple Response      | 0x0F             | Value Set                                                      |
| Read by Group Type Request  | 0x10             | Start Handle,Ending Handle,UUID                                |
| Read by Group Type Response | 0x11             | Length,Attribute Data List                                     |
| Write Command               | 0x52             | Attribute Handle,Attribute Value                               |
| Prepare Write Request       | 0x16             | Attribute Handle,Value Offset,Part Attribute Value             |
| Prepare Write Response      | 0x17             | Attribute Handle,Value Offset,Part Attribute Value             |


see. _bluetooth_core [Vol 3, Part F] Attribute Opcode Summary_
