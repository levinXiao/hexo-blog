---
title: 在UILabel中用富文本NSMutableAttributedString和NSTextAttachment实现最简单的图文混排
date: 2016-09-12 15:33:28
tags: [iOS,图文混排]
categories: iOS
---

在iOS6之后 UILabel API中添加了一个属性 attributedText

```
// the underlying attributed string drawn by the label, if set, the label ignores the properties above.
//如果设置了这个属性 label会一些其他的属性而直接使用这个属性来渲染文字 忽略的属性包括但不限于 text,font,textColor,shadowColor,shadowColor,shadowOffset,textAlignment,lineBreakMode

@property(nullable, nonatomic,copy)   NSAttributedString *attributedText NS_AVAILABLE_IOS(6_0);  // default is nil
```

现在要实现在UILabel中直接使用一些简单的图文混排 我们就是利用的这个属性来实现

```
  NSTextAttachment *attch = [[NSTextAttachment alloc] init];
  attch.image = [UIImage imageNamed:@"d_aini"];
  //设置图片大小
  attch.bounds = CGRectMake(0, 0, 32, 32);

  //创建带有图片的富文本
  NSAttributedString *string = [NSAttributedString attributedStringWithAttachment:attch];
  [attri appendAttributedString:string];

  // 用label的attributedText属性来使用富文本
  label.attributedText = attri;
```

NSMutableAttributedString的一些其他的用法

```
    // 创建一个富文本
    NSMutableAttributedString *attri = [[NSMutableAttributedString alloc] initWithString:@"哈哈哈哈哈123456789"];
    // 修改富文本中的不同文字的样式
    //设置前景色
    [attri addAttribute:NSForegroundColorAttributeName value:[UIColor blueColor] range:NSMakeRange(0, 5)];
    //设置字体
    [attri addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:20] range:NSMakeRange(0, 5)];
    // 设置数字为红色
    [attri addAttribute:NSForegroundColorAttributeName value:[UIColor redColor] range:NSMakeRange(5, 9)];
    //设置字体
    [attri addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:30] range:NSMakeRange(5, 9)];

```