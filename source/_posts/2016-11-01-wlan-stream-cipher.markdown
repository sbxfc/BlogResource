---
layout: post
title: "WLAN密钥解密"
date: 2016-11-01 16:10:22 +0800
comments: true
categories:
---

#解密操作

> 使用工具 Wireshark 和 Aircrack-ng

首先,打开 Wireshark 并监听无线网接口。如果已开启WIFI,这时候就可以看到许多杂乱无章的捕获数据。

我们进行一下过滤,通过过滤 Beacon帧来寻找WIFI热点(Access Point,简称AP)。Beacon 是由热点发出的,用于告知设备自己的存在的数据帧。Beacon 是管理帧 type=00 (即0x00),并且 subtype=1000(即0x08):


![ws](/images/2016/10/tmp75c54425.png)


选择一个WIFI热点,然后点击查看其帧信息,并记录下MAC地址。(看名称可知,这是一个苹果的网卡接口,是一台开着共享的Mac电脑):

![ws](/images/2016/10/tmp11983903.png)

接着,我们以这个MAC地址线索再寻找一个与其相连的设备。(这个设备是我的iPad,因为之前获得过这个WIFI的密码,所以我把iPad的完整MAC地址隐藏掉了)

![ws](/images/2016/10/tmp0a502955.png)

在得到设备和热点的MAC地址后,我们就可以通过一些工具尝试让他们解除认证,然后抓取他们重连时的认证信息。这里,我用了之前自己写的小工具:

<https://github.com/sbxfc/wlan-macos/tree/master/deauth>

	$ make
	$ ./deauth en1 -s xx:xx:xx:xx:b0:73 -a c8:e0:eb:58:34:bd --rate 1 - number 10

在运行上面的示例之前,要确保你的 Wireshark 正在处于抓包状态。执行后,如果发现捕获到了四次握手数据就可以停止Wireshark了,如果没有那就再来一次。

![ws](/images/2016/10/tmp2c8e7312.png)

在成功捕获握手数据之后,将捕获数据进行保存（Wireshark->文件->另存为）。然后打开 Aircrack-ng 对数据进行解密。 Aircrack-ng 实际上是通过现有的密码字典通过PRG算法生成密钥流与加密后的流进行一系列的比对,从而找到正确的密码值。也就是说,如果你的WiFi密码设置的越复杂也就越难以破解。

在使用 Aircrack-ng 对数据流进行解密时,我们需要一个密码字典,你可以从网上下一个常用的密码字典。在确定设备上安装  Aircrack-ng 之后,就可以在终端下开始解密了:

	aircrack-ng -w /Volumes/sbxfc/pwd_dic/破解字典/wordlist/wordlist.txt /Users/sbxfc/Downloads/tmp.pcap

在运行之后,程序会询问你选择哪一个目标,一般是第一个,后面标识 handshake。输入序列号,然后回车。由于我选择的WIFI热点密码太过简单,回车之后立刻就返回结果了:

![ws](/images/2016/10/tmp53ad4572.png)

#四次握手

802.11i 中的 RSN（Robust Security Network）定义了无线网络下的安全的连接流程。这个流程也就是 RSNA（Robust Security Network Association）,定义了无线网络下的认证、加密以及密钥管理。

STA 通过 802.1x 的认证后,AP 与 STA 都会拿到同一组 session key。有 RADIUS(认证服务器)时称为 PMK(Pairwise Master Key),无 RADIUS 时 PSK(Pre-Shared Key) 即PMK。之后进行 4-way handshake。

> 在没有RADIUS时,AP 与 STA 会预先设定好一组 passphrase,并用其衍生出PMK。

对于网络安全而言,key愈少愈好。4-way handshake 建立了 512 bits的PTK(Pairwise Transient Key)。PTK 由 PMK,AP Nonce,STA Nonce,AP'MAC,STA'MAC 生成。4-way handshake 也会产生GTK（Group Temporal Key）,用来解密 multicast和broadcast traffic。这个GTK是所有STA公用的一个key。

![4-handshake](/images/2016/10/tmp13152092.png)

1. AP 发送 ANonce 至 STA。
2. STA 收到后,用 ANonce 和已知信息生成PTK,并通过PTK中的KCK生成检验码MIC,附上SNonce 发送至 AP。
3. AP 收到后也生成 PTK,然后用PTK中的KCK部分对MIC进行校验。成功,则发送GTK和MIC至STA。
4. STA 答复。

WPA1 TKIP的 PTK 长度512bits,WPA2 CCMP的PTK长度为384bits。其中,TMK1 和 TMK2 只用在 TKIP 加密data时。

PTK由几部分组成:

内存区域 | 简称 | 长度(bits)| 名称 | 用途
------------ | ------------- | ------------| ------------
0-127   | KCK  |128 | EAPOL-Key Confirmation Key| 计算WPA EAPOL key message 的MIC
128-255  | KEK  |128| EAPOL-Key Encryption Key| 加密额外要送给STA的data,如 GTK or RSN IE
256-383  | TEK |128| Temporal Encryption Key| 解密unicast packets
384-447  | TMK1  |64| Temporal AP Tx MIC Key| 计算AP 发送的unicast packet的MIC
448-511  | TMK2  |64| Temporal AP Rx MIC Key| 计算STA 发送的unicast packet的MIC

PTK算法:

首先使用PBKDF2（Password-Based Key Derivation Function 2）算法生成一个32字节的PMK key，该算法需要执行4096*2轮,同时由于使用了SSID（0-32字符）进行salt。

	 PMK = PBKDF2(HMAC−SHA1, pwd, ssid, 4096, 256)

PTK使用PRF-512（pseudo random functions 512bits）算法产生，通过PMK、固定字符串、AP_Mac、Sta_Mac、ANonce、SNonce六个输入参数得到一个512 bits的PTK。
	 
	 PTK = PRF-512(PMK, “Pairwise key expansion”, Min(AP_Mac, Sta_Mac) ||Max(AP_Mac, Sta_Mac) || Min(ANonce, SNonce) || Max(ANonce, SNonce))

MIC算法:
	
	//WAP1
	MIC = HMAC(EVP_sha1(), KCK, 16, eapol_data，eapol_size) 
	//WAP2
	MIC = HMAC(EVP_md5(), KCK, 16, eapol_data，eapol_size)

#解密原理

![4-handshake](/images/2016/10/142303547147.png)

破解时,利用我们字典中 PSK 和 ESSID 生成PMK。

然后结合已知的STA'MAC、BSSID、AP'NONCE、STA'NONCE计算出PTK。

然后加上原始的报文数据算出MIC,并与AP发送的MIC比较。如果一致，那么该PSK就是密钥。


#参见

- <https://zh.wikipedia.org/wiki/%E6%B5%81%E5%8A%A0%E5%AF%86>
- <http://www.cnblogs.com/rainbowzc/p/5410876.html>
