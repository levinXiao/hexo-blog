---
title: macosx sierra安全性和隐私中找不到任何来源的解决办法
date: 2016-10-10 10:17:49
tags: [macOS]
categories:
---

安装完macos sierra后，如果看不到【任何来源】
在终端里输入：
```
sudo spctl --master-disable 
```
即可在安全选项中看到重新出现的允许任何来源选项