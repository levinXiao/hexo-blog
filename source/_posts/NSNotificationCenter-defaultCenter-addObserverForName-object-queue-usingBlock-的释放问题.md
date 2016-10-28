---
title: >-
  [[NSNotificationCenter defaultCenter]
  addObserverForName:object:queue:usingBlock:]的释放问题
date: 2016-08-31 15:47:52
tags: [oc,iOS]
categories: iOS
---


之前一直以为addObserverForName:object:queue:usingBlock的释放是和addObserver:selector:@selector()name:object:的释放处理逻辑是一样的

后来测试了一下 发现并不是这样
查看文档

```
- (id <NSObject>)addObserverForName:(nullable NSString *)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block NS_AVAILABLE(10_6, 4_0);
    // The return value is retained by the system, and should be held onto by the caller in
    // order to remove the observer with removeObserver: later, to stop observation.

```
apple说这个方法的返回值会被system retain一下 如果要为了移除这个观察者,必须自己保留其返回值 然后removeObserver调用

所以 正确的做法是

初始化

```
id<NSObject> notiObserver = [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationWillEnterForegroundNotification object:nil queue:nil usingBlock:^(NSNotification *note) {
        //doSomeing
}];
```
用全局变量将notiObserver保存 然后退出页面的时候去释放调

```
if (notiObserver) [[NSNotificationCenter defaultCenter] removeObserver:notiObserver];
```

这样就可以正确的释放了