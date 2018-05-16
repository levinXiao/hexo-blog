---
title: UITableViewHeaderFooterView 的使用和透明背景色的设置
date: 2016-08-06 21:14:52
tags: iOS,oc,UITableView
categories: iOS
---
####环境 iOS 9.3 下测试   2016-06-16

我们知道UITableView有代理方法设置每个section的headerview和footerview

为了减少UITableView的内存开销

<!-- more -->

Apple引入了UITableViewHeaderFooterView这个类 这个类是在iOS 6 的时候加入 和UITableViewCell这个类一样 这个类由reuseIdentifier 这个概念
所以初始化方法也和UITableViewCell类似了

直接上代码

###Apple 关于UITableViewHeaderFooterView的定义
```
#import <Foundation/Foundation.h>
#import <UIKit/UIView.h>
#import <UIKit/UITableView.h>

// Either the header or footer for a section
NS_ASSUME_NONNULL_BEGIN

NS_CLASS_AVAILABLE_IOS(6_0) @interface UITableViewHeaderFooterView : UIView

- (instancetype)initWithReuseIdentifier:(nullable NSString *)reuseIdentifier NS_DESIGNATED_INITIALIZER;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder NS_DESIGNATED_INITIALIZER;

@property (nonatomic, strong, null_resettable) UIColor *tintColor;

@property (nonatomic, readonly, strong, nullable) UILabel *textLabel;
@property (nonatomic, readonly, strong, nullable) UILabel *detailTextLabel; // only supported for headers in grouped style

@property (nonatomic, readonly, strong) UIView *contentView;
@property (nonatomic, strong, nullable) UIView *backgroundView;

@property (nonatomic, readonly, copy, nullable) NSString *reuseIdentifier;

- (void)prepareForReuse;  // if the view is reusable (has a reuse identifier), this is called just before the view is returned from the table view method dequeueReusableHeaderFooterViewWithIdentifier:.  If you override, you MUST call super.

@end

NS_ASSUME_NONNULL_END
```

###实际应用
```

- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section{
    if (tableView.tag == 100) {
        static NSString *tableViewHeaderViewIdentifier = @"tableViewHeaderViewIdentifier";
        UITableViewHeaderFooterView *headView = [tableView tableViewHeaderViewIdentifier];
        if (!headView) {
            headView = [[UITableViewHeaderFooterView alloc] initWithReuseIdentifier:tableViewHeaderViewIdentifier];
        }
        UILabel *tableHeaderLabel = [headView viewWithTag:10001];
        if (!tableHeaderLabel) {
            tableHeaderLabel = [[UILabel alloc]init];
            tableHeaderLabel.tag = 10001;
            tableHeaderLabel.frame = (CGRect){20,0,WIDTH(tableView)-40,40};
            tableHeaderLabel.font = Font(inchesForValue(16, 18));
            tableHeaderLabel.textColor = RGBColor(26, 161, 230);
            tableHeaderLabel.textAlignment = NSTextAlignmentCenter;
            [headView.contentView addSubview:tableHeaderLabel];
        }
        tableHeaderLabel.text = @"title";

        UIView *lineView = [headView viewWithTag:10002];
        if (!lineView) {
            lineView = [[UIView alloc] init];
            lineView.tag = 10002;
            lineView.frame = (CGRect){0,0,WIDTH(tableView),0.5};
            lineView.backgroundColor = RGBAColor(118, 134, 147, 0.2);
            [headView addSubview:lineView];
        }

        UIView *bottomLineView = [headView viewWithTag:10003];
        if (!bottomLineView) {
            bottomLineView = [[UIView alloc] init];
            bottomLineView.tag = 10003;
            bottomLineView.frame = (CGRect){0,HEIGHT(tableHeaderLabel)-0.5,WIDTH(tableView),0.5};
            bottomLineView.backgroundColor = RGBAColor(118, 134, 147, 0.2);
            [headView addSubview:bottomLineView];
        }
        return headView;
    }
    return nil;
}

```

这样设置之后 会发现apple给的默认的UITableViewHeaderFooterView的背景颜色是灰色
如果我们要改成透明颜色

这时候就要加入这个代理了

```
- (void)tableView:(UITableView *)tableView willDisplayHeaderView:(UIView *)view forSection:(NSInteger)section {
    if ([view isMemberOfClass:[UITableViewHeaderFooterView class]]) {
        ((UITableViewHeaderFooterView *)view).backgroundView.backgroundColor = [UIColor clearColor];
    }
}
```
