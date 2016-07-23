---
layout: post
title: "Objective-C runtime那些事儿"
date: 2016-06-15 23:44:34 +0800
comments: true
categories: 
---
runtime 号称iOS开发的黑魔法。

现在就让我来探究探究runtime，一来作为学习笔记，二来给有需要的人参考。

在使用runtime的时候，我们可能会看到一些奇怪的字段，现在就让我来一一讲解一下：

1：SEL

     OC在编译时，根据方法的名字生成一个用来区分这个方法的唯一ID，这些SEL组成了一个set集合，当我们在这个集合中查找某个方法时，只需去查找这个方法所对应的SEL即可。所以，SEL的本质就是字符串。
     
2：id

     id是一个指向objc_object结构体的指针。
3：isa

     objective_c中的object在最后编译的时候会被转成c的结构体，而在这个结构体中有一个isa指针，指向它的类别class。所以，isa是一个指向类别class的结构体指针。
     
4：meta class
     当我们发送一个消息给nsobject对象时，这条消息会在对象 的类的方法列表中查找。
   当我们发送一个消息给一个类时，这条消息会在类的meta class的方法列表中寻找。故meta class就是一个类对象的class。
   
5：IMP:

     本质就是一个函数指针，这个被指向的函数包含一个被接受消息的的对象id，调用方法的SEL以及一些方法的参数，并返回一个id，因此我们可以通过SEL获得它所对应的IMP，在取得了函数指针之后，也就意味着我们找到了需要执行方法的代码入口，这样我们就可以像普通的c语言函数调用一样使用这个行数指针。
6：Ivar:

     在object中被定义为：typedef struct objc_ivar，它是一个指向objc_ivar结构体的指针，结构体如下定义：
     
```
                  struct_objc_ivar{
                               char *ivar_name 
                               char *ivar_type
                               int ivar_offset
                  }
```


主要的理论就是这些，接下来就看看怎么使用者高深莫测的runtime了。假设我们有一个实体类Car，它有如下两个属性。

```
@property(nonatomic,copy)NSString *brand;
@property(nonatomic,copy)NSString *engine;
```

在我们给定这两个参数初始值之后，如果你在程序的运行过程中想修改它的值，那么我们就可以使用runtime来进行变量的控制：

```
-(void)changeCar{
    unsigned int number=0;
    __weak typeof(self) weakSelf=self;
    Ivar *ivar=class_copyIvarList([self.car class], &number);
    for (int i=0; i<number; i++) {
        Ivar var=ivar[i];
        const char *varName=ivar_getName(var);
        NSString *propertyName=[NSString stringWithUTF8String:varName];
        int y=arc4random()%7;
        if ([propertyName isEqualToString:@"_brand"]) {
            __strong typeof(weakSelf) strongSelf=weakSelf;
            object_setIvar(self.car, var, strongSelf.brandArray[y]);
            break;
        }
    }
    
    self.label.text=self.car.brand;
}

//数组
- (NSArray *)brandArray{
    if (!_brandArray) {
        _brandArray=@[@"Benz",@"Honda",@"BMW",@"Lamborghini",@"Ferrari",@"Porsche",@"audio"];
    }
    return _brandArray;
}
```

这里是随机动态改变Car的brand，从兰博基尼到东风轰达随机切换。
接下来就是使用runtime来动态添加方法：

  ```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    class_addMethod([self.car class], @selector(printParameter), (IMP)printParameter, "v@:");
    
    [self.button addTarget:self action:@selector(buttonAction) forControlEvents:UIControlEventTouchUpInside];
}

void printParameter(id self,SEL _cmd){
    NSLog(@"5.2v10自然进气发动机610匹马力4轮驱动，0-100加速3.2s");
   
}

-(void)printParameter{
    
}

-(void)buttonAction{
    if ([self.car performSelector:@selector(printParameter)]) {
        [self.car performSelector:@selector(printParameter)];
    }
}
```

以上代码表示每点击一次按钮就动态添加printParameter方法来输出当前Car的动力参数。

接下来就看看怎么使用runtime来拦截和交换方法：

     再引入plane实体，我们交换plane和car的参数输出函数，让汽车输出飞机的动力参数，代码如下：
     
```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.textView.text=@"ready? go";
 }

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}


- (IBAction)planePress:(id)sender {
    self.textView.text=[self.plane planePowerTakeOff];
}

- (IBAction)exchangePowerPress:(id)sender {
    Method carMethod=class_getInstanceMethod([self.car class], @selector(carPowerTakeOff));
    Method planeMethod=class_getInstanceMethod([self.plane class], @selector(planePowerTakeOff));
    method_exchangeImplementations(carMethod, planeMethod);
    self.textView.text=@"power is Exchanged";
}
- (IBAction)changePowerPress:(id)sender {
    Method carMethod=class_getInstanceMethod([self.car class], @selector(carPowerTakeOff));
    Method lannboMethod=class_getInstanceMethod([self class], @selector(changePower));
    method_exchangeImplementations(carMethod, lannboMethod);
     self.textView.text=@"car power is changed";
}

-(NSString *)changePower{
    return @"我是兰博基尼超跑，我的动力比飞机还强，我到香港只需8分钟";
}

- (IBAction)carPress:(id)sender {
      self.textView.text=[self.car carPowerTakeOff];
}
```

最后是使用runtime为方法添加新功能。

这里我们给按钮的点击事件触发的方法添加新的功能。所以增加UIButton的分类，在分类中写入以下代码：


```
+ (void)load{
    static dispatch_once_t onceManager;
    dispatch_once(&onceManager, ^{
        Class customerClass=[self class];
        SEL oriSEL=@selector(sendAction:to:forEvent:);
        Method oriMethod=class_getInstanceMethod(customerClass, oriSEL);
        
        SEL customerSEL=@selector(customerAction:to:forEvent:);
        Method customerMethod=class_getInstanceMethod(customerClass, customerSEL);
        
        BOOL isSuccess=class_addMethod(customerClass, oriSEL, method_getImplementation(customerMethod), method_getTypeEncoding(customerMethod));
        if (isSuccess) {
            class_replaceMethod(customerClass, customerSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
        }else{
            method_exchangeImplementations(oriMethod, customerMethod);
        }
    });
}
-(void)customerAction:(SEL)action to:target forEvent:(UIEvent *)event{
    NSLog(@"我是披着奔驰外观的众泰,众泰汽车，实现您的豪车梦，哈哈哈");
    [self customerAction:action to:target forEvent:event];
}
```

自此，每次点击button触发的监听方法都将输出众泰汽车的广告。

欧耶，runtime的常用功能就是这些，更高深的用法还在研究之中。最后，本着无demo不文章的精神，给出本篇文章我的github地址：https://github.com/voidxin/RunTimeDemo

感谢google上各位老司机的技术分享，本文更多的是对各位老司机分享的总结和实践。谢谢。