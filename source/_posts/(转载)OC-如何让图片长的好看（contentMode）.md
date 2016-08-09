---
title: (转载)OC-如何让图片长的好看（contentMode）
date: 2016-08-06 21:14:52
tags: [oc,iOS,图片处理]
categories: iOS
---
转载至CSDN http://blog.csdn.net/yi_zz32/article/details/50381762

我们在开发中，经常要在view，tableView，等显示图片，比如我们发微博（带有图片的），那么我们常常要考虑如何处理这些图片，是否拉伸，拉伸到什么样的效果等等，常常是需要考虑的问题
其实主要的还是要理解UIImageView的ContentMode的这些参数，这些参数一般就能满分我们的开发需求

```

 UIViewContentModeScaleToFill, 
 UIViewContentModeScaleAspectFit, // contents scaled to fit with fixed aspect. remainder is transparent 
 UIViewContentModeScaleAspectFill, // contents scaled to fill with fixed aspect. some portion of content may be clipped.   
 UIViewContentModeRedraw, // redraw on bounds change (calls -setNeedsDisplay) UIViewContentModeCenter, // contents remain same size. positioned adjusted. UIViewContentModeTop, 
 UIViewContentModeBottom, 
 UIViewContentModeLeft, 
 UIViewContentModeRight, 
 UIViewContentModeTopLeft, 
 UIViewContentModeTopRight, 
 UIViewContentModeBottomLeft, 
 UIViewContentModeBottomRight,
```

那我们接下来，就来说明一下，这些值都代表什么意思

```
UIViewContentModeScaleToFill：图片拉伸至填充这个UIImageView(图片可能变形)
UIViewContentModeScaleAspectFit : 图片拉伸至完全显示在UIImageView里面为止（图片不会变形）
UIViewContentModeScaleAspectFill ： 图片拉伸至 图片的宽度等于UIImageView的宽度 或者 图片的高度等于UIImageView的高度为止，然后将图片居中显示
UIViewContentModeRedraw ： 调用了setNeedsDisplay方法时，就会将图片重新渲染
UIViewContentModeCenter：居中显示

```