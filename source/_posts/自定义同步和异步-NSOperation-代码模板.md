---
title: 自定义同步和异步 NSOperation 代码模板
date: 2018-05-16 17:29:36
tags: [iOS,多线程]
categories: iOS
---

## 同步的 **NSOperation**
在实际情况中，定义同步的 **NSOperation** 比较简单，不需要显式的设置 **finish** 的值

**同步的 NSOperation 模板**

### .h
```
#import <Foundation/Foundation.h>

//非并发
@interface XXXOperation : NSOperation

@end
```
<!-- more -->

### .m

```

@interface XXXOperation ()

@end

@implementation XXXOperation

#pragma mark -
- (void)start {
    NSLog(@"main");
    @try {
        @autoreleasepool {
            //添加自动释放池 是因为 子线程拿不到主线程的自动释放池
            if ([self isCancelled]) {
                NSLog(@"======>>[operation] operation isCancelled YES");
                return;
            } else {
                NSLog(@"======>> [operation] operation start");
                NSInteger i = 0;
                while (i < 5){
                    NSLog(@"i : %ld",(long)i);
                    i++;
                }
            }
        }
    }
    @catch (NSException *exception) {
      NSLog(@"======>> [operation] exception catch");
    }
    NSLog(@"======>> [operation] operation finished");
}

- (void)cancel {
    [super cancel];
}

- (void)dealloc {
    [self timerTimeoutInvalid];
    NSLog(@"======>>[operation] operation:%p dealloc ",self);
}

@end
```



## 异步的 **NSOperation**

异步的 **NSOperation** 场景从字面上不是非常容易理解，简单来说：

```
我需要构建一个线程池，该线程池用来从网络下下载图片，考虑到性能的需求，我需要能够动态的设置线程池能同时下载图片的数量
现在假设这个数量默认为3个，即我需要开启三个 NSOperation 来下载图片
现在问题来了，下载图片这个过程显然使异步的，在同步的 NSOperation 中已经不能满足我们的需求
所以，异步的NSOperation就成了解决方案
```

从苹果的官方文档上，我们可以看出来 **NSOperation** 的结束是由 **finished** 方法来进行判定
在 **NSOperation** 中 **finished** 的getter方法是 **isFinished**

**下面是一个异步的 NSOperation 模板**

## .h
```
#import <Foundation/Foundation.h>

//非并发
@interface XXXOperation : NSOperation

@end
```

## .m

```

@interface XXXOperation ()

// getter=isFinished 必须指定，如果不指定需要 子类覆写 isFinished 方法
// getter=isExecuting 同上
@property (nonatomic, assign, getter=isFinished) BOOL finished;
@property (nonatomic, assign, getter=isExecuting) BOOL executing;

@end

@implementation XXXOperation

@synthesize finished = _finished;
@synthesize executing = _executing;

#pragma mark -
- (void)start {
    NSLog(@"main");
    @try {
        @autoreleasepool {
          //添加自动释放池 是因为 子线程拿不到主线程的自动释放池
            if ([self isCancelled]) {
                NSLog(@"======>>[operation] operation isCancelled YES");
                self.finished = YES;
                return;
            } else {
                self.executing = YES;
                NSLog(@"======>>[operation] operation starting...");
                @weakify(self);
                void (^block)() = ^(){
                    @strongify(self);
                    self.finished = YES;
                    self.executing = NO;
                };
                [self startOperationComplete:block];
            }
        }
    }
    @catch (NSException *exception) {
      NSLog(@"======>> [operation] exception catch");
      self.finished = YES;
      self.executing = NO;
    }
}

- (void)startOperationComplete:(void(^)())complete {
   //测试代码
    NSLog(@"======>> [operation] operation started");
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_current_queue(), ^{
       if (complete) complete();
    });
}

- (void)setFinished:(BOOL)finished {
    NSLog(@"======>> [operation] operation will finish");
    [self willChangeValueForKey:@"isFinished"];
    _finished = finished;
    [self didChangeValueForKey:@"isFinished"];
    NSLog(@"======>> [operation] operation finished");
}

- (void)setExecuting:(BOOL)executing {
    [self willChangeValueForKey:@"isExecuting"];
    _executing = executing;
    [self didChangeValueForKey:@"isExecuting"];
}

- (void)cancel {
    [super cancel];
    NSLog(@"======>> [operation] cancel");
    // 如果正在执行中则表示已经start过，可以将isFinished设为yes
    [self timerTimeoutInvalid];
    if (self.isExecuting) {
        self.finished = YES;
        self.executing = NO;
    }
}

- (void)dealloc {
    [self timerTimeoutInvalid];
    NSLog(@"======>>[operation] operation:%p dealloc ",self);
}

@end
```
