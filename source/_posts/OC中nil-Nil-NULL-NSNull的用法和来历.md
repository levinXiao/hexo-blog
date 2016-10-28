---
title: OC中nil/Nil/NULL/NSNull的用法和来历
date: 2016-09-08 11:06:55
tags: [iOS,oc]
categories: iOS
---

学过C的同学都知道，C是用0来表示不存在的原始值。而NULL作为空指针，在指针环境中也相当于0值。其实NULL和0的值都是一样的。但是为了目的和用途及容易识别的原因，NULL用于指针和对象，0用于数值。

# NULL
要理解NULL首先得知道这么几个概念：

#### 什么是空指针常量（null pointer constant）

```
An integer constant expression with the value 0, or such an expression cast to type void *, is called a null pointer constant.
```

值为0的整型常量表达式或者类型为void*的表达式则称为空指针常量。

#### 什么是空指针（null pointer）

```
If a null pointer constant is converted to a pointer type, the resulting pointer, called a null pointer, is guaranteed to compare unequal to a pointer to any object or function.
```

当一个空指针常量被转换为指针类型。这个指针不指向任何实际的对象或者函数，则这个指针就称为空指针。

举个例子：
char *p = 0; 此时p就是一个空指针。它不指向任何实际对象。反过来说：任何实际的对象和函数的地址都不可能是空指针。

#### 空指针（null pointer）指向了内存的什么地方（空指针的内部实现）

标准并没有对空指针指向内存中的什么地方这一个问题作出规定，也就是说用哪个具体的地址值（0x0 地址还是某一特定地址）表示空指针取决于系统的实现。常见的空指针一般指向 0 地址，即空指针的内部用全 0 来表示（zero null pointer，零空指针）；在OC中空指针指向的地址为0x0。测试代码如下：

 char *p = 0;
 printf("%p\n", p); // 0x0
了解了上面的几个概念后我们再来看一看NULL的定义

在头文件stddef.h中可以找到这么一句定义：

```
#define NULL ((void*)0)
```

由定义和上面的几个概念可以看出NULL是一个值为0的空指针（本人认为），即可以这么认为：这个指针指向的地址为0x0。也就是说这个指针不指向任何对象和函数。


# nil

nil是Objective-C在C的表达不存在的基础上增加的。

nil是一个指向不存在的对象指针。也就相当于NULL，虽然两者语义上不同，但是技术上是相等的。刚被分配内存的NSObject的内容都被设置为0，即都为空指针。

**nil有一个特别的行为就是，它虽然为零，仍然可以向他发送消息。在nil上调用方法都返回一个零值.**

``` 
NSString *name;
NSLog(@"%p  %hhd", name, [name isEqualToString:@"CoderKo1o"]); // 0x0 0
```


# Nil
在苹果官方文档中，可以看到这么一段定义

```
#define nil __DARWIN_NULL
#define Nil __DARWIN_NULL

```

由此可见：Nil和nil值是一样的，那么他们有什么区别呢？

```
nil : Defines the id of a null instance.

Nil : Defines the id of a null class.

```

nil是指向0值的对象**指针**， Nil是指向0值的**类指针**，即就是为了用来区分是对象指针和类指针。
# NSNull
按照上面的说法，每个东西都有存在的意义，当然NSNull也有自身存在的价值。NSNull在Foundation和其他框架中被广泛地使用。

熟悉NSArray和NSDictionary之类的集合都知道，它们有一个nil值的缺陷，因为语法规定了这些集合是以nil为结束标志的。所以当你想存储一个值为空的对象。又不能使用nil（语法规定），这时候NSNull就派上用场了，NSNull可以理解为有效地对NULL或者nil值封装成一个对象。

在Foundation框架中的NSNull.h头文件中可以发现NSNUll只有一个类方法+ (NSNull *)null;返回一个表示0值的单独的对象,注意:这个对象指向的地址并不为0x0。

总结
这里介绍的都是OC中用来表达没有的值，也是在学习OC过程中不可忽视的,为了更直观地表现这4个的值区别，这里列了一个表：

标志 | 值 | 含义
--- | -- | ----
NULL  |	(void *)0 |	C中的空指针（值为0）
nil	| (id)0	| Objective-C对象的空指针
Nil	| (Class)0	| Objective-C类的空指针
NSNull	| [NSNull null] | 用来表示零值的单独的对象（并非空指针）

