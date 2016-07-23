---
layout: post
title: "使用UIActivityViewController实现AirDrop分享"
date: 2016-05-08 14:41:51 +0800
comments: true
categories: 
---
今天闲来无事，无意中看了一篇利用AirDrop实现文件传输的文章，于是动手写了一个很简单的demo。其实就是使用UIActivityViewController。
具体实现如下：

```
#import "DemoViewController.h"
@interface DemoViewController ()
@property(nonatomic,strong)UIWebView *webView;

@end

@implementation DemoViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor=[UIColor whiteColor];
    
    [self addUI];
    [self loadDataFile];
   
}
-(void)loadDataFile{
    NSURL *fileURL=[self returnURLWithFileName:@"demo.png"];
    [self.webView loadRequest:[NSURLRequest requestWithURL:fileURL]];
}
-(void)addUI{
    //add webView
    [self.view addSubview:self.webView];
    //add rightBarItem
     self.navigationItem.rightBarButtonItem=[[UIBarButtonItem alloc]initWithTitle:@"分享" style:UIBarButtonItemStyleDone target:self action:@selector(sharedAction)];
}
- (UIWebView *)webView{
    if (!_webView) {
        _webView=[[UIWebView alloc]init];
        _webView.frame=self.view.bounds;
    }
    return _webView;
}
#pragma mark -return URL for fileName
-(NSURL *)returnURLWithFileName:(NSString *)fileName{
    NSArray *arrs=[fileName componentsSeparatedByString:@"."];
    NSString *pathStr=[[NSBundle mainBundle] pathForResource:arrs.firstObject ofType:arrs[1]];
    NSURL *fileURL=[NSURL fileURLWithPath:pathStr];
    return fileURL;
    
    
}

#pragma mark rightBarItem Action
-(void)sharedAction{
    NSLog(@"rightBarItem is clicked");
    NSURL *fileURL=[self returnURLWithFileName:@"demo.png"];
    NSArray *urls=@[fileURL];
    UIActivityViewController *activituVC=[[UIActivityViewController alloc]initWithActivityItems:urls applicationActivities:nil];
    NSArray *cludeActivitys=@[UIActivityTypePostToFacebook,
                               UIActivityTypePostToTwitter,
                               UIActivityTypePostToWeibo,
                               UIActivityTypePostToVimeo,
                               UIActivityTypeMessage,
                               UIActivityTypeMail,
                               UIActivityTypeCopyToPasteboard,
                               UIActivityTypePrint,
                               UIActivityTypeAssignToContact,
                               UIActivityTypeSaveToCameraRoll,
                               UIActivityTypeAddToReadingList,
                               UIActivityTypePostToFlickr,
                               UIActivityTypePostToTencentWeibo];
    activituVC.excludedActivityTypes=cludeActivitys;

    [self presentViewController:activituVC animated:YES completion:nil];

}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```
详细的demo可以访问我的github去下载，希望可以一起交流一起进步：
demo下载地址：https://github.com/voidxin/AirDropDemoWithZX