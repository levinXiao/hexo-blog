---
title: iOS 中 copy 和 mutableCopy 以及 one-level-deep-copy 进阶
date: 2018-02-01 15:29:20
tags: [iOS,oc]
categories: iOS
---

对 copy 和 mutableCopy 的概念还有一些不了解的同学，可以看看这篇文章
<https://www.jianshu.com/p/700f58eb0b86>

有了上面的了解之后，我们看看这段代码

```
NSMutableArray *eleInMutable = [NSMutableArray arrayWithObject:@"1"];
//二维数组
NSMutableArray *array1 = [NSMutableArray arrayWithObject:eleInMutable];
//
NSMutableArray *array2 = array1.mutableCopy;
//像二维数组的第一个元素数组添加一个2
[array2[0] addObject:@"2"];
    
printPointer(array1);
/* print result :  0x10070ec50 */
printPointer(array2);
/* print result :  0x10070ec80 */
``` 

在这里我们可以看到 pointer 的结果却是是不一样的。

**但是！！！！！！！**

如果我们打印 array1 和 array2 的结果

```
printArray(array1);
/* print result :  [["1","2"]] */
printArray(array2);
/* print result :  [["1","2"]] */
```

会惊奇的发现 array1 的结果 和 array2 的结果完全一样

**这不科学，我一定是眼花了！！！！！！**

不死心的我打印了 array1 和 array2 这两个二维数组的子数组的指针

```
printPointer(array1[0]);
/* print result :  0x10070ee40 */
printPointer(array2[0]);
/* print result :  0x10070ee40 */
```

**居然完全一样，居然，会是，一样！！！！！！！！**

## one-level-deep copy

查看了[苹果官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Collections/Articles/Copying.html#//apple_ref/doc/uid/TP40010162-SW3)后，找到了对应的说明

```
There are two ways to make deep copies of a collection. 
You can use the collection’s equivalent of initWithArray:copyItems: with YES as the second parameter. 
If you create a deep copy of a collection in this way, 
each object in the collection is sent a copyWithZone: message. 
If the objects in the collection have adopted the NSCopying protocol, 
the objects are deeply copied to the new collection, 
which is then the sole owner of the copied objects. 
If the objects do not adopt the NSCopying protocol, 
attempting to copy them in such a way results in a runtime error. 
However, copyWithZone: produces a shallow copy. 
This kind of copy is only capable of producing a one-level-deep copy. 
If you only need a one-level-deep copy, 
you can explicitly call for one as in Listing 2
```

发现了一个新的名词**one-level-deep copy**这个就是本文讨论的主题了

## one-level-deep copy 是什么

one-level-deep copy 从字面上理解，可以理解为单层深复制。
那 deep copy 我们就换一种说法，叫做完全复制。

从苹果的官方文档来看，在 ***类集合元素*** 进行 mutableCopy 的时候，编译器会自动的进行单层深复制。在***非类集合元素*** 进行 mutableCopy 的时候，编译器就会进行完全复制了。

那这样就解释的通本文开头所展示的代码了。

## 类集合元素的完全复制

那如果我们在实际项目的使用中，如果要对类集合元素进行完全复制，苹果官方文档也给出了答案

方法1：

```
NSArray *deepCopyArray=[[NSArray alloc] initWithArray:someArray copyItems:YES];
```

当 **copyItems** 参数为 **YES** 的时候，会进行完全复制。
同理，当 **copyItems** 参数为 **NO** 的时候，会进行单层深复制。

方法2：

```
NSArray* trueDeepCopyArray = [NSKeyedUnarchiver unarchiveObjectWithData:
          [NSKeyedArchiver archivedDataWithRootObject:oldArray]];

```

注意：要使用 NSKeyedUnarchiver 序列化对象要对对象实现 NSCoding 协议，当然，Foundation 库的 **类集合元素（NSArray及其子类,NSSet及其子类,NSDictionary及其子类）** 都实现了 NSCoding 协议


## 结语

没有结语，只有代码

[完整项目下载](http://o7b4rtbje.bkt.clouddn.com/copytest.zip)

示例代码：

```
NSMutableArray *eleInMutable = [NSMutableArray arrayWithObject:@"1"];
//二维数组
NSMutableArray *array1 = [NSMutableArray arrayWithObject:eleInMutable];
//
NSMutableArray *array2 = array1.mutableCopy;
//像二维数组的第一个元素数组添加一个2
[array2[0] addObject:@"2"];

//开始验证 mutablecopy 是否是深复制
//如果是深复制 那么 array2[0]中添加元素不会影响 array1的结果
printPointer(array1);
/* print result :  0x10070ec50 */
printPointer(array2);
/* print result :  0x10070ec80 */
    
printArray(array1);
/* print result :  [["1","2"]] */
printArray(array2);
/* print result :  [["1","2"]] */
    
    
printPointer(array1[0]);
/* print result :  0x10070ee40 */
printPointer(array2[0]);
/* print result :  0x10070ee40 */
```




