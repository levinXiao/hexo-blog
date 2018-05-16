---
title:  iOS9使用提示框的正确实现方式(UIAlertView is deprecated)
date: 2016-08-06 21:14:52
tags: [iOS,oc]
categories: iOS
---
#前言

在从iOS8到iOS9的升级过程中，弹出提示框的方式有了很大的改变，在Xcode7 ，iOS9.0的SDK中，已经明确提示不再推荐使用UIAlertView，而只能使用UIAlertController，我们通过代码来演示一下。
我通过点击一个按钮，然后弹出提示框，代码示例如下：

<!-- more -->
```
#import "ViewController.h"  

@interface ViewController ()  

@property(strong,nonatomic) UIButton *button;  

@end  

@implementation ViewController  

- (void)viewDidLoad {  
  [super viewDidLoad];  

  self.button = [[UIButton alloc] initWithFrame:CGRectMake(0, 100, [[UIScreen mainScreen] bounds].size.width, 20)];  
  [self.button setTitle:@"跳转" forState:UIControlStateNormal];  
  [self.button setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];  
  [self.view addSubview:self.button];  

  [self.button addTarget:self action:@selector(clickMe:) forControlEvents:UIControlEventTouchUpInside];  

}  

-(void)clickMe:(id)sender{  

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"提示" message:@"按钮被点击了" delegate:self cancelButtonTitle:@"确定" otherButtonTitles:nil, nil nil];  
    [alert show];  

}  

@end  


```

##需要注意的是
######在iOS 9.0 SDK 环境下编译上述代码时，会有下列的警告提示：

“‘UIAlertView’ is deprecated:first deprecated in iOS 9.0 - UIAlertView is deprecated. Use UIAlertController with a preferredStyle of UIAlertControllerStyleAlert instead”

意思是UIAlertView首先在iOS9中被弃用（不推荐）使用。让我们去用UIAlertController。但是运行程序，发现代码还是可以成功运行，不会出现crash。

解决这个warning，使用UIAlertController来解决这个问题。代码如下

```
//初始化提示框；  
  UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"提示" message:@"按钮被点击了" preferredStyle:  UIAlertControllerStyleAlert];  

  [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {  
    //点击按钮的响应事件；  
  }]];  

  //弹出提示框；  
  [self presentViewController:alert animated:true completion:nil];  

```
这样，代码就不会有警告了。

程序运行后的效果同上。  

其中preferredStyle这个参数还有另一个选择UIAlertControllerStyleActionSheet

选择这个枚举类型后，发现这个提示框是从底部弹出的。通过查看代码还可以发现，在提示框中的按钮响应不再需要delegate委托来实现了。直接使用addAction就可以在一个block中实现按钮点击，非常方便。

#总结
可以发现这里我们呈现一个对话框使用了presentViewController这个方法，这个方法是呈现模态视图(Modal View)的方法，也就是是说，此时的提示框是一个模态视图。当我们在进行界面跳转的时候，也一般使用这个方法，此时呈现的第二个ViewController也是一个模态视图。我们可以把模态视图理解为一个浮动在原先视图上的一个临时性的视图或者界面，当在模态视图中调用dismissViewController方法时，会返回上一个界面，并销毁这个模态视图对象。
