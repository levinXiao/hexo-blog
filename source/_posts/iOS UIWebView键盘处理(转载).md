---
title: iOS UIWebView键盘处理(转载)
date: 2017-04-17 21:14:52
tags: [iOS,oc,键盘]
categories: iOS
---
转载自:
http://blog.csdn.net/assholeu/article/details/38714123

如果你有下面的问题，此文也许会帮到你。

1. 键盘遮盖了UIWebView。
2. 如何拖动UIWebView来移除键盘。
3. 键盘出现时UIWebView里面的Content内容向上移动，以至聚焦的文本框超出了UIWebView的可视区域。
4. 如何在键盘弹出时禁止UIWebView里面的Content向上移动。
5. 无法在UIWebView中获取到坐标，来计算contentOffset得到想要展示的结果。

<!-- more -->

##一步一步说明：

#####1.  唤出移除键盘
只要点击UIWebView里面的html文本框控件，会自动弹出键盘。当然你需要获取键盘的信息（高度等），方法还是使用UIViewController+Notification的方式，代码如下：
```
// UIKeyboardWillShowNotification和UIKeyboardWillHideNotification为键盘弹出或移除时iOS系统post notification的名字，这里只需要定义self为这个通知的接收者即可。  
    // viewWillAppear:和viewWillDisappear:大家应该都很清楚，这两个方法分别在self loadView和removefromsuperview后执行。  
    // 特别注意：这里的object参数需要是nil,不然取不到键盘的userInfo  
    - (void)viewWillAppear:(BOOL)animated {  
        [super viewWillAppear:animated];  
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillShowNotification object:nil];  
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillHide:) name:UIKeyboardWillHideNotification object:nil];  

    }  

    - (void)viewWillDisappear:(BOOL)animated {  
        [super viewWillDisappear:animated];  
        [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardWillShowNotification object:nil];  
        [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardWillHideNotification object:nil];  
    }  

    - (void)keyboardWillShow:(NSNotification *)notification {  
        NSDictionary *userInfo = [notification userInfo];  
        NSValue* value = [userInfo objectForKey:UIKeyboardFrameEndUserInfoKey];  
        CGRect keyboardRect = [value CGRectValue]; // 这里得到了键盘的frame  
        // 你的操作，如键盘出现，控制视图上移等  
    }  

    - (void)keyboardWillHide:(NSNotification *)notification {  
        // 获取info同上面的方法  
        // 你的操作，如键盘移除，控制视图还原等  
    }  
```

#####2. 通过拖动UIWebView来移除键盘

```

self.webView.scrollView.keyboardDismissMode = UIScrollViewKeyboardDismissModeOnDrag; // 当拖动时移除键盘  

```
如果是iOS7以下，请参照 6 来设置，大概思路，先添加一个private的flag表明现在键盘是否存在，当存在时，通过 6 来获取事件关闭键盘。

#####3. 键盘遮盖了UIWebView
这个的解决方法可在 1 中的keyboardWillShow:里面操作，通过改变webView的origin来实现。
#####4. 键盘出现时UIWebView里面的Content内容向上移动，以至聚焦的文本框超出了UIWebView的可视区域
在UIWebView中，只要键盘出现，UIWebView肯定会向上移动，至于合不合适就不好说了，如果不合适，就只用禁用自动移动。

#####5. 如何在键盘弹出时禁止UIWebView里面的Content向上移动
这个方法，我也找了很久，但是还是找到了，感谢强大的网友，代码如下：
```
@interface XXX : UIViewController<UIScrollViewDelegate> // 添加UIScrollViewDelegate， step 1  

    self.webView.scrollView.delegate = self; // 注册代理， step 2  

    - (UIView*)viewForZoomingInScrollView:(UIScrollView*)scrollView{ // 实现代理方法， step 3  
            return nil;  
    }
```

#####6. 如何在UIWebView中获取点击坐标
众所周知，UIWebView会吃掉所有的touch事件，不然也不会有那么多人费工夫弄javascript了，但是不能设置不代表不能以另外一种方式代替，大概思路：给webView的superView添加手势，然后通过实现多手势过滤设置来实现，为什么要设置多手势过滤呢？我这里说明一下，由于UIWebView默认有自己的手势，它会拦截掉你的手势，以至 superView无法接收手势，代码如下：

```
@interface XXX : UIViewController<UIGestureRecognizerDelegate> // 添加UIGestureRecognizerDelegate， step 1  

    // 添加手势， step 2  
    UITapGestureRecognizer *webTap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(webTap:)];  
    webTap.numberOfTouchesRequired = 1;  
    webTap.numberOfTapsRequired = 1;  
    webTap.delegate = self;  
    webTap.cancelsTouchesInView = NO;  
    [self.view addGestureRecognizer:webTap];  

    // 设置过滤，ruturn YES为同时接收，至此手势可以透过webView，让你的superView也可以接收到了， step 3  
    -(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer{  
            return YES;  
    }  

    - (void)webTap:(UITapGestureRecognizer *)sender{  
            CGPoint tapPoint = [sender locationInView:self.webView.scrollView]; // 获取相对于webView中的坐标，如果改成self.view则获取相对于superView中的坐标， step 4  
            NSLog(@"tapPoint x:%f y:%f",tapPoint.x,tapPoint.y);  
    }  
```

UIWebView键盘处理能想起的就只有这些了，欢迎大家补充。
转载自:
http://blog.csdn.net/assholeu/article/details/38714123

资料参考：
感谢 http://blog.csdn.net/abel_tu/article/details/12134261
