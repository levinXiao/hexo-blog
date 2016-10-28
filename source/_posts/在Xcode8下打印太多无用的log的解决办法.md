---
title: 在Xcode8下打印太多无用的log的解决办法
date: 2016-09-22 11:43:53
tags: [iOS]
categories: iOS
---

升级了Xcode8后 运行之前的项目,发现log会多出很多之前没有出现过的信息,发现是一些系统的打印的信息,很苦恼

后来发现是Xcode8的新特性

解决办法:

在scheme中添加环境变量 
```
OS_ACTIVITY_MODE = disable
```

* 选中 ***target***下的 ***Edit Scheme***
* 在弹出的窗口中 左侧 ***Run*** 一栏的右边选中 ***Arguments***
* 在 ***Environment Variables***中添加环境变量 ***OS_ACTIVITY_MODE = disable***
* 点击 ***Close***

重新编译运行

发现 世界清净了