---
title: Cocoa Mac OS X下打开应用程序的OC代码
date: 2017-01-03 21:14:52
tags: [macOS,oc]
categories: macOS
---
```
NSTask *softTask = [[NSTask alloc] init];

[softTask setLaunchPath:@"/Applications/Xcode.app/Contents/MacOS/Xcode"];

[softTask launch];
```

<!-- more -->
