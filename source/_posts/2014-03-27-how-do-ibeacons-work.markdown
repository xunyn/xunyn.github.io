---
layout: post
title: "How do iBeacons work"
date: 2014-03-27 20:15
comments: true
categories: techincal
---
[原文地址](http://www.warski.org/blog/2014/01/how-ibeacons-work/)
##iBeacons
iBeacons最近是一个趋势的话题，它们允许室内定位，让你的电话知道你在基站的范围。这个能有许多应用：在停车场帮你找到你的车，零售商通过优惠券和基于位置的特别优惠，以至许多现在不敢想象的应用。  
![iBeacons](http://xunyanan.com/images/blog/iStock_000019319211Large-300x187.jpg)
关于iBeacons和如何使用它有许多博文，但从技术角度，他们如何工作，深层的技术是低功耗蓝牙，等等。。。  
##什么是低功耗蓝牙
[低功耗蓝牙](http://en.wikipedia.org/wiki/Bluetooth_low_energy)是发布在2010年的蓝牙4.0规范的一部分，它在Nokia发起于2006年，但合并到蓝牙技术，它是同经典蓝牙不同的一组协议，并且设备不向后兼容。因此有会遇到三种设备
*蓝牙：仅支持经典模式
*智能蓝牙过渡(Bluetooth Smart Ready):支持经典模式和LE(Low Energy)模式
*智能蓝牙：仅支持LE模式  
![Bluetooth Smart Ready](http://xunyanan.com/images/blog/LogoBluetoothSmartReady-300x130.jpg)
</br>  
新的智能手机（iPhone 4S+,SG3+）,笔记本，平板电脑，装备了蓝牙4.0，因为已经是智能蓝牙过渡，Beacons，从另一方面讲，仅支持低功耗协议(允许他们在一块电池的情景下，工作足够长的时间)，因此他们实现了智能蓝牙，旧的设备，如外设、车载系统、旧手机通常仅支持经典蓝牙协议。  
低功耗蓝牙主要关注点，理所当然在低功耗上。例如一些Beacons在一块纽扣电池下（电池通常不更换，除非当beacon停止工作时，才需要更换）可以持续两年传输信号。低功耗蓝牙同经典蓝牙都使用相同的频谱范围（2.4GHZ--2.4835GHZ）。低功耗蓝牙有更低的传输速率，尽管它的本意不是流传输大量数据，而是发现和简单通讯。在理论范围，低功耗蓝牙和经典蓝牙信号都可以最远达到100米。  
##低功耗蓝牙如何通讯  
低功耗蓝牙有两种主状态:广播和连接。    
广播是单向的被发现机制，想要被发现的设备可以传输数据包每20ms到10秒。越短的时间间隔，越短的电池寿命，但设备发现速度越快。数据包可以加长至47个长度的字节，由以下构成:    
* 1字节报头  
* 4字节访问地址  
* 2-39字节广播频道PDU  
* 3字节CRC    
![data](http://xunyanan.com/images/blog/bluetooth-le-packet-300x121.png)  
<!--more-->
广播通讯频道，访问地址总是0x8E89BED6.对于数据频道，每一次连接都不相同。  
PDU有它自己的头部（两字节：负载的大小和它的类型--设备是否支持连接，等）和实际的数据负载（最长至37字节）。  
最后，负载的头六个字节是设备的MAC地址，实际的数据最长有31字节。  
低功耗蓝牙可以操作一个非连接广播模式（所有的信息都被包含在广播中），但他们也可以允许连接（通常允许）。  
设备被发现之后，可以建立一个连接。它是有可能去读低功耗蓝牙提供的服务，对于每一个服务特性（这个是被熟知的GATT配置文件的实现）。每一个特性提供一些可以被读取或被写的值。例如一个智能调温器可以暴露服务为获得当前温度、湿度读取（作为特性服务），另一个服务特性去设置期望的温度。尽管，beacons没有使用连接，这里跳过了细节。如果你想要读更多的低功耗蓝牙连接，[苹果核心蓝牙指导书](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257-CH1-SW1),即使你不是一名iOS开发人员，也能提供了一个好的概述。更专业的技术文章，请看[EE](http://www.eetimes.com/document.asp?doc_id=1280724).    
##beacons如何使用蓝牙？  
Beacons通常使用广播频道，由beacon望文生义，他们每隔一段时间传输数据包，这些数据被设备像智能机接受，因此iBeacons是简单的低功耗蓝牙广播的特殊应用，iOS端加入了特别的支持。  
一国你尝试拦截一个蓝牙广播包，例如来自于Estimote beacon，你将看到下列数据：  
`
02 01 06 1A FF 4C 00 02 15 B9 40 7F 30 F5 F8 46 6E AF F9 25 55 6B 57 FE 6D 00 49 00
`  
(捕获这些数据，如果使用OSX，一个附加的[下载](http://stackoverflow.com/questions/5863088/bluetooth-sniffer-preferably-mac-osx)包含一个蓝牙扫描和包日志)  
以上数据已经有报文了，固定访问地址、广播PDU头和MAC地址被移除;它只有广播数据--30字节，因此它适应了31字节的限制。  
什么使得蓝牙广播成为iBeacon？格式由苹果规定，分解如下：  
`
02 01 06 1A FF 4C 00 02 15: iBeacon prefix (fixed)
B9 40 7F 30 F5 F8 46 6E AF F9 25 55 6B 57 FE 6D: proximity UUID (here: Estimote’s fixed UUID)
00 49: major
00 0A: minor
C5: 2’s complement of measured TX power
`  
![data](http://xunyanan.com/images/blog/bluetooth-le-ibeacon-packet-300x144.png)  
如果你想用beacons试验，不需要任何特别的设备，遵循一下步骤：如果你有一台新手机（例如iPhone4s + SG3+）或者蓝牙4.0的笔记本（Retina Macbook），你可以转化以上设备成为iBeacon的发送和接受端，对于iPhone，参考“Locate iB”app在苹果应用商店，对于MacOS，[参考这里](http://stackoverflow.com/questions/19410398/turn-macbook-into-ibeacon),当然，也可以用[电路板](http://developer.radiusnetworks.com/2013/10/09/how-to-make-an-ibeacon-out-of-a-raspberry-pi.html)当beacon。    
##分解iBeacon数据格式  
从固定的iBeacon前缀(02 01 ....15),其他的部分的意义是什么呢？  
近距离UUID(B9 ... 6D 在我们的例子中)，是被用来区分你的beacons和其他的标识符，如果，例如beacons被用来去提供特别优惠给顾客在连锁店。所有的beacons属于一个链，应该有相同的近距离UUID，专用的为连锁店的iPhone应用，会在后台扫描给定UUID的beacons。  
主编号(2字节，这里是:0x0049 (10进制73))，被用来为相关的许多beacons组团，例如在店里的所有的beacons将有相同的主标号，这样应用将知道顾客在哪个特定的商店。  
次标号(2字节，这里是:0X000A,(10进制10))被用来定义独立的beacons，每一个beacon在店里将有不同的此标号，因此你知道顾客实际在哪里。  
##测量距离  
数据最后，TX power,被用来判断离beacon的距离。这些可以被表达做为粗糙的信息（近/远/超出范围）或者作为一个以米更精确的度量(可以转换为英尺);这是如何完成的？  
TX power(这里是：OxC5=197，2‘s complement = 256-197 =-59dBm)是被测量的信号强度，在距设备一米(RSSI-接受信号强度指示)，作为信号衰减预期，更进一步的讲，知道一米的RSSI，并且知道当前的RSSI(我们同接受信号一起得到信息)，它有可能有计算差别，iOS已经建立了模型，对于其他平台，需要手写代码，[参看这里](http://stackoverflow.com/questions/20416218/understanding-ibeacon-distancing)  
障碍物例如家具，人或者通讯拥堵都可以降低信号，因此距离是一个估算值。  
##iOS 整合  
iOS同iBeacons带来了一些附加的整合，你的app可以接受通知，当一个beacon出现在一个区域--不仅是app在前台运行，后台运行也可以。一个app可以订阅区域的进入/离开时间，然后它可以部分的唤醒，甚至它还未在工作，为了响应这些事件，app可以发送本地通知，提示用户打开app，看店内提示(可以从网络取数据)，或者相关内容。    
更准确的，当手机不活跃时，iOS进入低功耗监控模式：仅有iBacons区域进入和退出是被检测的。当手机和app是活跃的，你可以进入搜寻模式，可以打开你去家册信号强度和更精确估算距离。  
注意你的手机需要花费时间去检测beacon，首先beacons传输广播是间断的，其次你的手机不是活跃的，它也是隔一段时间检测蓝牙信号，对于被检测的beacon，这两种间隔必须重合，实际上，它可能花费15分钟去检测一个beacon。
对于一步一步写一个iOS iBeacons的应用，[参考这里](http://www.cocoanetics.com/2013/11/can-you-smell-the-ibeacon/),Beacons制造商也提供专用的SDK，去帮助编写基于beacon的引用，看一下例子[Esitmote's iOS SDK](https://github.com/Estimote/iOS-SDK)and[Android](https://github.com/Estimote/Android-SDK)  
##下一步
现在唯一的剩下去做的事是开发一些基于beacon的引用。
