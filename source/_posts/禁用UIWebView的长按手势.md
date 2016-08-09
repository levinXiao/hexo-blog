---
title: 禁用UIWebView的长按手势
date: 2016-08-06 21:14:52
tags: [iOS,oc]
categories: iOS
---
当webview显示html页面的时候 本身会提供一些原生的交互行为

长按文字选中就是其中的一种

这时候 如果要禁用这个效果

在webview的delegate中加入如下的代码

```
- (void)webViewDidFinishLoad:(UIWebView *)webView{
    NSLog(@"webViewDidFinishLoad");
    
    [webView stringByEvaluatingJavaScriptFromString:@"document.documentElement.style.webkitUserSelect='none';"];
    [webView stringByEvaluatingJavaScriptFromString:@"document.documentElement.style.webkitTouchCallout='none';"];
}

```

当然  更好的方法是在html的原始文件中 加入如下的代码

```
 <script type="text/JavaScript">

    window.onload=function({
      document.documentElement.style.webkitTouchCallout='none';
};
 </script>
```

这样子不管谁调用都没有这个手势了