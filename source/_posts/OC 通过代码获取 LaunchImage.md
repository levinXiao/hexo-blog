---
title: OC 通过代码获取 LaunchImage
date: 2016-08-06 21:14:52
tags: [oc,LaunchImage,iOS]
categories: iOS
---
```
    CGSize viewSize =self.view.bounds.size;
    NSString *viewOrientation = @"Portrait";//横屏请设置成 @"Landscape"
    NSString *launchImage = nil;
    NSArray *imagesDict = [[[NSBundle mainBundle] infoDictionary] valueForKey:@"UILaunchImages"];
    
    for(NSDictionary *dict in imagesDict) {
        CGSize imageSize =CGSizeFromString(dict[@"UILaunchImageSize"]);
        if(CGSizeEqualToSize(imageSize, viewSize) && [viewOrientation isEqualToString:dict[@"UILaunchImageOrientation"]]) {
            launchImage = dict[@"UILaunchImageName"];
        }
    }
```