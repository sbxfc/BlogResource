---
layout: post
title: "IEEE 802.11 WLAN 网路协议"
date: 2016-06-23 15:27:41 +0800
comments: true
categories: 
---

#IEEE 802.11

IEEE 802.11 是现今无线局域网的通讯标准,该标准包括物理层和数据链路层中的媒体访问控制层（MAC层）的两部分规范。

#物理层

IEEE 802.11 中的物理层由 PLCP Sublayer、PMD Sublayer 以及 Physical-Layer Management 三部分组成。PLCP主要进行载波的分析和针对不同的物理层形成的相应格式分组。PMD层用于识别相关介质传输的信息所使用的调制和编码技术。物理管理层为不同的物理层进行信道选择和协调。

#MAC层


![IEEE_802_11](/images/2016/6/ieee80211.png)

MPDU(Mac protocal data unit)即IEEE 802.11在MAC层数据帧格式。802.11在MAC层的数据帧分为三大类,类型由Frame control里的Type决定,每个大类又会根据Subtype的不同分为许多子类。首先是三大类:

- 00,管理帧 这类数据帧主要用来进行身份验证,发送热点信号(Beacon)等。

- 01,控制帧 发送RTS/CTS,ACK等一些查询和控制响应帧。

- 10,数据帧 携带更高层的数据(如IP数据包，ISO7层协议)。

##1,管理帧

管理帧根据Subtype类型不同,分为如下类型:

管理类掩码| 子类型掩码  | 帧描述
------------ | ------------- | ------------
00 |0000 |Association request (连接请求) 
00 |0001 |  Association response (连接响应)  
00 |0010 | Reassociation request（重连接请求）
00 |0011 | Reassociation response（重连接联响应）
00 |0100 | Probe request（探测请求）
00 |0101 | Probe response（探测响应）
00 |1000 | Beacon（信标，被动扫描时AP 发出，notify）
00 |1001 | ATIM（通知传输指示消息）
00 |1010 | Disassociation（解除连接，notify）
00 |1011 | Authentication（身份验证）
00 |1100 |Deauthentication（解除认证，notify）


1000 Beacon(信标)的帧格式:

![IEEE_802_11](/images/2016/6/ieee80211_beacon.jpeg)


##2,控制帧 

类型信息| 子类型信息  | 帧描述
------------ | ------------- | ------------
01 |1010 |Power Save（PS）- Poll（省电－轮询）
01 |1011 |RTS（请求发送，即: Request To Send ，预约信道，帧长20字节）
01 |1100 |CTS（清除发送，即:Clear To Send ，同意预约，帧长14字节）
01 |1101 |ACK（确认）
01 |1110 |CF-End（无竞争周期结束）
01 |1111 |CF-End（无竞争周期结束）＋CF-ACK（无竞争周期确认）

##3,数据帧 

类型信息| 子类型信息  | 帧描述
------------ | ------------- | ------------
10 | 0000 | Data（数据）
10 | 0001 | Data+CF-ACK
10 | 0010 | Data+CF-Poll
10 | 0011 | Data+CF-ACK+CF-Poll
10 | 0100 | Null data（无数据：未传送数据）
10 | 0101 | CF-ACK（未传送数据）
10 | 0110 | CF-Poll（未传送数据）
10 | 0111 | Data+CF-ACK+CF-Poll
10 | 1000 | Qos Data
10 | 1001 | Qos Data + CF-ACK
10 | 1010 | Qos Data + CF-Poll
10 | 1011 | Qos Data + CF-ACK+ CF-Poll
10 | 1100 | QoS Null（未传送数据）
10 | 1101 | QoS CF-ACK（未传送数据）
10 | 1110 | QoS CF-Poll（未传送数据）
10 | 1111 | QoS CF-ACK+ CF-Poll（未传送数据）



#RadioTap 

当网络接口处于monitor模式时,内核会生成一个RadioTap数据头添加在MPDU前面,Radiotap记录了热点的信息,如信号强度、MPDU帧信息等信息。

RadioTap包括 Header 和 Data两部分数据,其中Header的结构如下:

	struct ieee80211_radiotap_header {
        u_int8_t        it_version;     /* set to 0 */
        u_int8_t        it_pad;
        u_int16_t       it_len;         /* entire length */
        u_int32_t       it_present;     /* fields present */
	} __attribute__((__packed__));

- it_version 版本号,始终为0.
- it_pad 未使用,只作字段对齐使用。
- it_len 整个Radiotap的长度,如果你不关心Radiotap里包含的信息,可以通过这个值跳过Radiotap部分。
- it_present 是Data数据部分的掩码,标识哪些数据出现在接下来的Data里。

it_present里的每一位代表一种类型的数据是否出现在接下来的Data数据里,并且这些数据的出现的顺序是与掩码的位次依次相关的。其中前22位为标准掩码,其位置和对应的数据结构信息可以在这里查看:

- <http://www.radiotap.org/defined-fields>

通常it_present的最后一位Ext为0,此时Data紧随it_present之后出现。如果Ext为1,表明开发者增加了it_present字段,每个增加的it_present大小为32bit,直到最后一个Ext为0的it_present出现时,Data才会紧接着出现。除Ext之外Present中倒数第2、3位是设备厂商的保留位，不能作为Data掩码使用。

	.... .... .... .... .... .... .... ...1 = TSFT : True
	.... .... .... .... .... .... .... ..1. = Flags: True
	.... .... .... .... .... .... .... .1.. = Rate : True
	.... .... .... .... .... .... .... 1... = Channel : True
	.... .... .... .... .... .... ...0 .... = FHSS : False
	.... .... .... .... .... .... ..1. .... = Antenna signal  : False
	.... .... .... .... .... .... .0.. .... = Antenna noise : False 
	.... .... .... .... .... .... 0... .... = Lock quality : False
	.... .... .... .... .... ...0 .... .... = TX attenuation : False 
	.... .... .... .... .... ..0. .... .... = dB TX attenuation : False 
	.... .... .... .... .... .0.. .... .... = dBm TX power : False  
	.... .... .... .... .... 1... .... .... = Antenna  : True
	.... .... .... .... ...0 .... .... .... = dB antenna signal : False
	.... .... .... .... ..0. .... .... .... = dB antenna noise : False
	.... .... .... .... .1.. .... .... .... = RX flags : False
	.... .... .... .... 0... .... .... .... = MCS : False
	.... .... .... ...0 .... .... .... .... = A-MPDU status : False
	.... .... .... ..0. .... .... .... .... = VHT : False
	.... .... .... .0.. .... .... .... .... = ... 
	0... .... .... .... .... .... .... .... = Ext:False

掩码对应的数据信息

 位置编码| 大小 |名称| 描述
------------  | ------------- | ------------- |------------
0  |  8byte   | TSFT | 只对接收帧有效,表示MPDU第一个bit到达MAC时的时间,单位是微秒
1  |  1byte   | Flags| 发送或接收帧属性,包括一些有用信息,比如FCS是否符合 
2  |  1byte   | Rate | 传输或接收速率,单位500kbs
3  |  2byte   |  Channel | 发送接收信号的频率，单位是MHz
4  |  1byte   |  FHSS | 跳频技术，是无线通讯最常用的扩频方式之一
5  |  1byte   | Antenna signal | 天线的射频信号强度，单位是dBm
6  |  1byte   | Antenna noise  | 天线的射频噪声强度，单位是dBm；
7  |  2byte   | Lock quality | 信号质量
8  |  2byte   | TX attenuation | 与出厂标准最大功率相比的功率衰减，0为最大功率
9  |  2byte   | dB TX attenuation | 与出厂标准最大功率相比的功率衰减dB值，0为最大功率
10 |  1byte   | dBm TX power | 传输功率的dBm值，这是在天线端口测量的功率绝对值
11 |  1byte   | Antenna | 发送或接收该帧的天线索引编号（硬件编号从0开始）；
12 |  1byte   | dB antenna signal | 天线的射频信号的相对功率强度dB值；
13 |  1byte   | dB antenna noise | 天线的射频噪音的相对功率强度dB值；


#示例

- <https://github.com/sbxfc/wlan-sniffer/>

#参考

- <https://zh.wikipedia.org/wiki/IEEE_802.11>
- <http://www.radiotap.org/>
- <http://www.radiotap.org/defined-fields>