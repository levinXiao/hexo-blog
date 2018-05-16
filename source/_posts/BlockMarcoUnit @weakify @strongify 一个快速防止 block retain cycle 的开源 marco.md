---
title: BlockMarcoUnit @weakify @strongify 一个快速防止 block retain cycle 的开源 marco
date: 2018-02-05 16:31:59
tags: [iOS,oc]
categories: iOS
---

项目下载地址 [https://github.com/levinXiao/BlockMacroUnit]

## 循环引用（retain cycle）的由来和一般解决方案

在block语句块中，如果需引用self，而self对象中又持有block对象，就会造成循环引用循环引用(retain cycle)，导致内存泄露，比如以下代码

```objective-c
self.block = ^{
	[self dosomething];
};
```
<!-- more -->

一般我们是这么解决的

```objective-c
__weak typeof(self) weakSelf = self;
self.block = ^{
	[weakSelf dosomething];
};
```

这样虽然可以在编译时使 Xcode 不报错
但是在运行时，也可能会因为 self 的释放导致 **dosomething** 这个方法不能够执行。
要解决上述问题，需要在 block 内部将 weakself 重新变为强引用，这样 self 的释放才会在执行完 block 后被正确的释放

```obejctive-c
__weak typeof(self) weakSelf = self;
self.block = ^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    [strongSelf dosomething];
};}

```

## 为什么使用 BlockMarcoUnit

在实际使用中，如果在每个 block 前后都添加这两句会显得比较繁琐
在查看了 RAC 中相关的源码后，发现有一种简便的方法可是使用这种特性

```obejctive-c
#import "BlockMarco.h"

@weakify(self)
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    @strongify(self)

    [self doSomething];
    [self doOtherThing];
});
```

暂时就说这么多吧，如果有什么问题欢迎随时在 github 上 issue 或者 email xiaoamani@qq.com
