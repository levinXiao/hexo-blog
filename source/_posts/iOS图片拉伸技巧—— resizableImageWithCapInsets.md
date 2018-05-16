---
title: iOS图片拉伸技巧—— resizableImageWithCapInsets
date: 2017-05-12 21:14:52
tags: [iOS,oc,图片处理]
categories: iOS
---
原先以为很简单的东西，到了实际做的时候，才发现这里出错那里不对。浪费很多时间，究根结底，还是没有弄清楚文档。

在iOS5, UIImage添加了可以拉伸图片的函数，即：
```
 [UIImage resizableImageWithCapInsets:(UIEdgeInset){#top#,#left#,#bottom#,#right#}]
```

<!-- more -->

UIEdgeInset 是一个结构体 定义如下:
```
Creates and returns a new image object with the specified cap insets.

Discussion

You use this method to add cap insets to an image or to change the existing cap insets of an image. In both cases, you get back a new image and the original image remains untouched.

During scaling or resizing of the image, areas covered by a cap are not scaled or resized. Instead, the pixel area not covered by the cap in each direction is tiled, left-to-right and top-to-bottom, to resize the image. This technique is often used to create variable-width buttons, which retain the same rounded corners but whose center region grows or shrinks as needed. For best performance, use a tiled area that is a 1×1 pixel area in size.
```

上左下右4参数定义了cap inset，就是离四条边的距离。拉升时，cap到边的部分不会被拉升，其余部分则会被拉升。尤其需要注意的时，拉升的时候，是从左到右，从上到下的方向。通俗点说，拉升不是全方向的拉升，而是垂直和水平拉升的叠加。

![1](http://o7b4rtbje.bkt.clouddn.com/popover_background_left.png)

开始我设置参数{20,10,10,10}，在图上的位置大致：
![2](http://o7b4rtbje.bkt.clouddn.com/imageWrong.png)

这样拉升的结果：

![3](http://o7b4rtbje.bkt.clouddn.com/4.png)

很奇怪是不是，为什么出现了两个箭头（红色部分是设置的背景色用语区分）？再回头看下文档，才恍然大悟：

1.拉升的时候，是按前文说的两个方向来拉升
2.拉升的部分，是以tiled方式，简单的说就是以镜像的方式

按照1的规则，拉升的时候，水平和垂直方向都需要拉升。这样在水平拉升的时候，箭头其实处于拉升的部分。而拉升的时候，先按照原有的尺寸添加进去，不足的地方再把中间不拉升的部分填充进去，周而复始，直到填充完毕。因此，就有上面的现象了。

要达到需要的效果，必须按照如下的设置：

![4] (http://o7b4rtbje.bkt.clouddn.com/imageRight.png)

于是得到了我们需要的效果：

![5](http://o7b4rtbje.bkt.clouddn.com/5.png)

说实话，这个函数在iOS5 beta的时候就知道了，可是一直是不正确的理解。直到今天需要用到的时候，才发现一直没理解对。于此同时，也发现自己 局限在工作相关的部分，工作以外的东西不是光知道就可以，还是需要去实践的。否则，就会遇到今天的情形，被个小问题，折磨了好久。
