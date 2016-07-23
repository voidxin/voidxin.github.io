---
layout: post
title: "MVVM模式初体验（使用ReactiveCocoa获取网络数据)"
date: 2016-06-15 16:06:46 +0800
comments: true
categories: 
---
使用RAC也有一段时间了，由于此前的项目都是使用的MVC模式，网络请求都封装在固定的模块中，抽取出来十分不方便，所以到目前为止并没有涉及到使用RAC去做获取网络请求的情景。

  近期，着手重构目前手上的项目，准备给臃肿的Controller瘦身，MVVM貌似是一个不错的选择（既然使用了RAC，那为什么不使用MVVM呢?)。于是，开始上手体验MVVM模式和RAC的结合使用（RAC的基础知识在这就不介绍了，百度一下，一大推老司机都有分享）。不用不知道，一用才发现RAC和MVVM简直是绝配啊，那体验真是改变了我对编程的传统观念（函数响应式编程真心好用）。
  
  这里用一个小demo来举例：
  
  进入页面加载数据显示。这里使用我当前项目中的一个接口来模拟数据。既然是MVVM，那Model和ViewModel肯定是少不了的。如下：我们新建一个ViewModel叫LoadStoreViewModel(因为这里是加载商店数据),传统的MVC模式中，网络请求都是在ViewController中完成的，这里我们把网络请求封装到对应的ViewModel中去，能够有效的减少ViewController的负担，降低耦合性。
  
  LoadStoreViewModel主要有三个属性：statues(网络加载状态），code1（编号），以及保存数据的数组dataArry，还有一个加载数据信号loadDataSignal；
  
 初始化loadDataSignal：(主要进行网络请求）
 
```
//
- (RACSignal *)loadDataSignal{
    if (_loadDataSignal==nil) {
        AFHTTPRequestOperationManager *manager=[AFHTTPRequestOperationManager manager];
        manager.requestSerializer=[[AFJSONRequestSerializer alloc]init];
        NSDictionary *params=@{@"code1":@"MWG08A09"};
        _loadDataSignal=[manager rac_GET:kLoadURL parameters:params];
    }
    return _loadDataSignal;
}
```

调用loadDataSignal处理网络请求结果：

```

-(void)initWithSubscrible{
    [[self.loadDataSignal deliverOn:[RACScheduler mainThreadScheduler]] subscribeNext:^(RACTuple *jsonDataResult) {
        //请求成功，加载数据
        NSDictionary *tuple=[jsonDataResult objectAtIndex:0];
        NSArray *resultList=tuple[@"resultList"];
        if (resultList.count>0) {
            self.dataArray=[[[resultList.rac_sequence
                              map:^id(NSDictionary *dataSource) {
                              NSDictionary *dic=[(NSDictionary *)dataSource mutableCopy];
                                  WGStoreModel *model=[WGStoreModel mj_objectWithKeyValues:dic];
                                  return model;
                            }] array] mutableCopy];
        }
    }];
    
    //请求失败
    [self.loadDataSignal subscribeError:^(NSError *error) {
       self.statues=@"没有网络，哈哈";
    }];
}
```

ViewModel的操作完成，接下来要在Controller中绑定ViewModel
绑定ViewModel，初始化，然后监听ViewModel中的网络请求状态，获得ViewMode中网络请求结果，在Controller中给出相应的提示，数据加载成功，显示数据刷新控件：

```
-(void)bindViewModel{
    @weakify(self);
    self.storeViewModel=[[LoadStoreViewModel alloc]init];
    self.isLoading=YES;
    self.code1=kCode1;
    RAC(self.storeViewModel,code1)=RACObserve(self, code1);
    
    //加载状态
   [RACObserve(self, isLoading) subscribeNext:^(id x) {
        UIApplication.sharedApplication.networkActivityIndicatorVisible = [x boolValue];
   }];
    
    //加载网络数据成功
    [[[RACObserve(self.storeViewModel, dataArray) ignore:nil] doNext:^(id x) {
        self.isLoading=YES;
    }] subscribeNext:^(id x) {
        @strongify(self);
        self.isLoading=NO;
        //刷新控件--
        [self.tableView reloadData];
    }];
    
    //加载网络数据失败
    [[RACObserve(self.storeViewModel, statues) filter:^BOOL(id value) {
        //filter是过滤
        return value !=nil;
    }] subscribeNext:^(NSString *str) {
        UIAlertView *alertView=[[UIAlertView alloc]initWithTitle:@"提示" message:str delegate:self cancelButtonTitle:@"confirm" otherButtonTitles:nil, nil];
        [alertView show];
    }];
    
}
```

ViewController中初始化TableView；

```
- (UITableView *)tableView{
    if (!_tableView) {
        _tableView=[[UITableView alloc]initWithFrame:CGRectZero style:UITableViewStylePlain];
        _tableView.delegate=self;
        _tableView.dataSource=self;
        _tableView.showsVerticalScrollIndicator=NO;
        _tableView.rowHeight=49;
    }
    return _tableView;
}
```

tableView的数据源和协议方法实现

```
#pragma mark -tableView DataSource
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return 1;
}

-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return [self.storeViewModel.dataArray count];
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    static NSString *indefier=@"CELL";
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:indefier];
    if (!cell) {
        cell=[[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:indefier];;
    }
    WGStoreModel *model=self.storeViewModel.dataArray[indexPath.row];
    cell.textLabel.text=model.shopName;
    
    return cell;
}
```

无demo不文章，为了更好的理解本篇文章，请到我的github下载对应的demo，一运行探究竟(https://github.com/voidxin/ReactiveCocoaRequestData)

谢谢。