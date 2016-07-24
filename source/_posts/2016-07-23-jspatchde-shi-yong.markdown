---
layout: post
title: "JSPatch的使用"
date: 2016-07-16 00:18:22 +0800
comments: true
categories: 
---
JSpatch的更多用法可以去github上找文档。不多解释。
先来个简单的demo：
<!--more-->
1：先百度“JSPatch”，进入如下页面：

![首页@2x.png](http://upload-images.jianshu.io/upload_images/1376067-e4b6df49a3f1a642.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2：注册账号，添加APP项目：

![创建app.png](http://upload-images.jianshu.io/upload_images/1376067-6606bbb5b71b47ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3：获得appkey

4： 下载 SDK 并导入项目

下载 SDK 后解压，将JSPatch.framework
拖入项目中，勾选 "Copy items if needed"，并确保 "Add to target" 勾选了相应的 target。

![SDK1](http://upload-images.jianshu.io/upload_images/1376067-248f00cf2e4e0970.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5：在appdelegate中加入以下代码：

```
#import <JSPatch/JSPatch.h>
@implementation AppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [JSPatch startWithAppKey:@"你的AppKey"];
    [JSPatch sync];
    ...
}
@end
```

6:在viewController中添加一个TableVIew，写入以下代码:

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    [self addUI];
   // [self dataBegin];
}

-(void)addUI{
    [self.view addSubview:self.tableView];
    [self.tableView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view);
        make.right.equalTo(self.view);
        make.top.equalTo(self.view);
        make.bottom.equalTo(self.view);
    }];
}
#pragma mark -tableView Datasource
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return 1;
}
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return 3;
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    static NSString *indefier=@"CELL";
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:indefier];
    if (!cell) {
        cell=[[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:indefier];
    }
    cell.textLabel.text=[NSString stringWithFormat:@"%ld",indexPath.row];
    
    return cell;
}

#pragma mark tableView delegate

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    
    
}

#pragma mark -getter
- (UITableView *)tableView{
    if (!_tableView) {
        _tableView=[[UITableView alloc]initWithFrame:CGRectZero style:UITableViewStylePlain];
        _tableView.delegate=self;
        _tableView.dataSource=self;
    }
    return _tableView;
}

```

运行程序，控制台输出：

```
2016-07-16 00:30:06.074 JSPatchHotFixDemo[3309:137704] JSPatch: runScript
2016-07-16 00:30:06.086 JSPatchHotFixDemo[3309:137704] JSPatch: evaluated script, length: 992
2016-07-16 00:30:06.087 JSPatchHotFixDemo[3309:137704] JSPatch: request http://q.jspatch.com/a368c7abdb625542/1.0.0?v=1468600206.087138
2016-07-16 00:30:06.296 JSPatchHotFixDemo[3309:137899] JSPatch: request success {
    v = 2;
}
```

表明集成JSPatch成功。

运行app，我们发现页面是这样的：


![QQ20160715-1@2x.png](http://upload-images.jianshu.io/upload_images/1376067-e508240f0c90aa7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们希望重写-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath方法改变cell的文字，然后增加cell的高度。

7：创建main.js文件，在文件中写入以下代码：

```
require('VXJspatchTestViewController');
 require('UIColor');
defineClass("ViewController", {
            
            tableView_cellForRowAtIndexPath: function(tableView, indexPath) {
            var cell = tableView.dequeueReusableCellWithIdentifier("cell")
            if (!cell) {
            cell = require('UITableViewCell').alloc().initWithStyle_reuseIdentifier(0, "cell")
            }
            cell.textLabel().setText("去吧皮卡丘");
            return cell
            },
            tableView_heightForRowAtIndexPath: function(tableView, indexPath) {
                 return 100;
            },
            //instance method definitions
            tableView_didSelectRowAtIndexPath: function(tableView, indexPath) {
            var testVC = VXJspatchTestViewController.alloc().init();
            testVC.setTitle("我是JSPatch增加的方法");
            self.navigationController().pushViewController_animated(testVC, YES);
           
            }
            }, {});
```

如果你不会写JSPatch代码也没关系，百度“JSPatch Conventor",写入OC源码，自动翻译。

8：创建版本号，上传main.js文件：

![QQ20160715-2@2x.png](http://upload-images.jianshu.io/upload_images/1376067-d19f63678b06f210.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

9：ok，发布成功，退出app，重新启动运行，恭喜你，实现动态更新，tableViewcell的两个datasource和一个delegate方法已被重写（如果你发现你的代码没起作用，请退出重新启动多次）：

![QQ20160716-0@2x.png](http://upload-images.jianshu.io/upload_images/1376067-4c669b741182430e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击cell，push到如下页面：

![QQ20160716-1@2x.png](http://upload-images.jianshu.io/upload_images/1376067-e9a8d5a9ebf3b6de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

自此，我们实现了一个简单的动态更新，如果以后你的项目想在发版之后修复bug的话，目前使用JSpatch是比较流行的选择。

无Demo不文章，欢迎访问我的github下载本文的Demo：https://github.com/voidxin/JSPatchHotFixDemo