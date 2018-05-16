---
title: iOS真机抓包命令rvictl
date: 2017-10-21 18:04:01
tags: [iOS,网络抓包]
categories: iOS
---

iOS 5后，apple引入了RVI remote virtual interface的特性，它只需要将iOS设备使用USB数据线连接到mac上，然后使用rvictl工具以iOS设备的UDID为参数在Mac中建立一个虚拟网络接口rvi，就可以在mac设备上使用tcpdump，wireshark等工具对创建的接口进行抓包分析了。

![rvictl命令](http://o7b4rtbje.bkt.clouddn.com/EBEF47FF-E71F-4C02-B291-3FEF753F8A45.png)

<!-- more -->

这种流量分析方法，是直接捕捉的iOS设备上的网络流量，因此无论是wifi还是2G/3G等其他网络类型都可以捕捉到，一根USB数据线就搞定了，不需要苛刻的要求PC与iOS设备处于同一个网段，或必须越狱等局限了

* 1.将iOS设备使用usb连接到mac上
* 2.获取设备的udid,自行百度
* 3.创建rvi接口
		$ rvictl -s ($UDID)
* 4.使用wireshark或者tcpdump来进行抓包
* 5.结束后,移除rvi端口
		$ rvictl -x ($UDID)
