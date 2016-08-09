---
title: UITableView的Header不悬浮不吸附的方法
date: 2016-08-06 21:14:52
tags: [iOS,oc,UITableView]
categories: iOS
---
当 UITableView 的 style 属性设置为 Plain 时，这个tableview的section header在滚动时会默认悬停在界面顶端。

```

#pragma mark - UIScrollViewDelegate
//使用这个方法主要是 让tableview的section不要停留在界面上
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    CGFloat sectionHeaderHeight = 30;
    if (scrollView.contentOffset.y<=sectionHeaderHeight&&scrollView.contentOffset.y>=0) {
        scrollView.contentInset = UIEdgeInsetsMake(-scrollView.contentOffset.y, 0, 0, 0);
    } else if (scrollView.contentOffset.y>=sectionHeaderHeight) {
        scrollView.contentInset = UIEdgeInsetsMake(-sectionHeaderHeight, 0, 0, 0);
    }
}

- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{
    CGFloat sectionHeaderHeight = 30;
    if (scrollView.contentInset.top < 0 &&
        scrollView.contentInset.top < fabs(sectionHeaderHeight) &&
        scrollView.contentOffset.y <= sectionHeaderHeight+0.5) {
        [UIView animateWithDuration:0.3f animations:^{
            [scrollView setContentOffset:(CGPoint){0,0}];
            scrollView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0);
        }];
    }
}

- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView{
    CGFloat sectionHeaderHeight = 30;
    if (scrollView.contentInset.top < 0 &&
        scrollView.contentInset.top < fabs(sectionHeaderHeight) &&
        scrollView.contentOffset.y <= sectionHeaderHeight+0.5) {
        [UIView animateWithDuration:0.3f animations:^{
            [scrollView setContentOffset:(CGPoint){0,0}];
            scrollView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0);
        }];
    }
}

```