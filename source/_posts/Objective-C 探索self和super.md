---
title: Objective-C 探索self和super
date: 2018-07-27 18:28:52
tags: [oc,runtime]
categories: iOS
---


## 从问题出发

相信每个iOSer都碰到过这个问题

```objective-c
@interface Son : Father
@end

@implementation Son
- (instancetype)init{
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```
以上代码的打印结果

<!-- more -->

首先先公布下答案

```objective-c
Son
Son
```
按照面向对象的思想应该是分别打印 **Son** 和 **Father**

但是实际结果两句代码都打印的是 **Son** 这个是为什么呢

## self 和 super

self是类的隐藏参数，指向当前调用方法的对象。而super并不是如我们所想是指向当前对象父类的指针。其实super是一个编译器标识符，在运行时中与self相同，指向同一个消息接受者，只是self会优先在当前类的methodLists中查找方法，而super则是优先从父类中查找。

```
$ clang -rewrite-objc main.m
```

可以看到运行时代码 如下（去掉无用代码且简化后）

```
objc_msgSend((id)self, sel_registerName("class"));

objc_msgSendSuper((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("Son"))}, sel_registerName("class"));
```

```objc_msgSend``` 函数中的```self```参数是指指向接收消息的类的实例的指针，即消息接受者，而```op```参数则是指处理该消息的selector；```objc_msgSendSuper```函数中的参数```super```则是一个```objc_super```结构体，objc_super结构体定义如下：

```
/// Specifies the superclass of an instance. 
struct objc_super {
    /// Specifies an instance of a class.
    __unsafe_unretained id receiver;

    /// Specifies the particular superclass of the instance to message. 
    __unsafe_unretained Class super_class;

    /* super_class is the first class to search */
};
```

其中```receiver```是指类的实例，```super_class```则是指该实例的父类。可以看到在编译后的C++代码中有个```__rw_objc_super```结构体：

```
struct __rw_objc_super { 
    struct objc_object *object; 
    struct objc_object *superClass; 
    __rw_objc_super(struct objc_object *o, struct objc_object *s) : object(o), superClass(s) {} 
};
```

通过```(__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("Son"))}```该段代码可知：

我们把```self```以及```Son```的父类通过结构体的构造方法构造了一个```__rw_objc_super```结构体，也就是```objc_super```。

因此```objc_super```结构体中的 **receiver** 就是 **self** 。所以```[self class]```和```[super class]```指向的是同一个消息接受者，只是```self```会优先从当前类的实现中寻找方法处理消息，而```super```则是会优先从```objc_super```结构体中的```super_class```也就是父类的实现中查找。**Father** 及**Son** 中均未实现```- (Class)class;```方法，因此会逐级向上查找最终调用基类```NSObject```的```- (Class)class;```方法。

既然消息接受者是```self```，而```[self class]```和```[super class]```指向的是同一个消息接受者，因此该段代码均打印 ```Son```。
