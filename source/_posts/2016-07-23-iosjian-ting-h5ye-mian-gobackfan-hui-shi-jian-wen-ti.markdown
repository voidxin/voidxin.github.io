---
layout: post
title: "iOS监听H5页面goBack返回事件问题"
date: 2016-03-20 20:39:57 +0800
comments: true
categories: 
---
从native直接push到一个webView页面，隐藏navigationBar，使用H5的头部为导航栏。此时会出现一个问题，就是push出的这个webview没有了原生的navigationBar，那么在点击H5页面上的返回按钮时怎么pop到之前的页面呢？
<!--more-->
  当然，我们可以使用上一遍博客提到的利用webViewjavascriptBridge的第三方来解决，这就需要H5和nativ相配合，如果h5是另一个团队做的，那么解决这么一个简单的问题确实显得有点小题大做。
  
  所以可以使用一种更简单的方法，利用webView 的delegate方法来控制pop到H5页面还是Native页面。
  
  由于系统尚且要兼容iOS7，加上还涉及到native向webView写cookie的问题，而WKWebView貌似只支持在iOS8以上使用，在cookie处理和缓存处理方面会有比较大的坑，所以项目中依旧使用的是UIWebView。在webView的代理方法中添加如下代码：
  
```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    if (navigationType==UIWebViewNavigationTypeBackForward) {
        self.webView.canGoBack?[self.webView goBack]:[self.navigationController popViewControllerAnimated:YES];
    }
    return YES;
}
```

  这样就可以区分返回native还是H5




补充：
      以上适用整个页面都是H5的页面
      如果只有navigationBar是原生的，此时应该重写pop事件：如下
      
```
-(void)navigationBarItemBackImage{
    UIImage *image=[UIImage imageNamed:@"navigationBack"];
    if ([PSBTools systemVersion]>=7) {
        image= [image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
    }
    self.navigationItem.leftBarButtonItem=[[UIBarButtonItem alloc]initWithImage:image style:UIBarButtonItemStyleDone target:self action:@selector(goBackAction)];
}
-(void)goBackAction{
    if (self.webView.canGoBack==YES) {
        [self.webView goBack];
    }else{
        [self.navigationController popViewControllerAnimated:YES];
    }
}
```