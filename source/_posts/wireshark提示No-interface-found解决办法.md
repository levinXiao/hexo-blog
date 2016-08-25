---
title: wireshark提示No interface found解决办法
date: 2016-08-24 18:05:50
tags: [网络抓包]
categories: 网络
---

wireshark提示”No interface found”解决办法 

```
$ sudo chmod 777 /dev/bpf*
```

再重启wireshark即可

![wireshark-No-interface-found](http://o7b4rtbje.bkt.clouddn.com/70A5099F-882E-4521-8A5A-31FA8511445E.png)