---
layout: post
title: "七牛云储存实现iOS企业版分发"
date: 2016-07-22 11:39:18 +0800
comments: true
categories: 
---
iOS企业证书的发版这里就不多说了。
由于app现在放在公司自己的服务器中，为了实现https的plist下载链接还特意把plist挂在了开源中国上。小公司，带宽小，多用户升级的时候给公司服务器带来不小的压力。所以最近工头有把ipa挂在七牛上去的想法，这样就可以有效的缓解服务器的压力。
企业版分发的plist下载链接必须是https协议的，所以需要SSL证书。如果你公司没有证书，那么可以使用七牛。
接下来开始步骤：

1：使用xcode打包生成ipa文件（过程不多说）

2：登陆七牛（默认你已有账号，没账号请注册）

3：登陆之后添加资源：

![添加资源.png](http://upload-images.jianshu.io/upload_images/1376067-5594fd1b12180d34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4：创建空间：

![创建空间.png](http://upload-images.jianshu.io/upload_images/1376067-8010026c145506ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5：创建成功之后在资源列表中你可以看到你新创建的资源:

![资源列表.png](http://upload-images.jianshu.io/upload_images/1376067-9ca45a61ed93db17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6：选中你新创建的资源名称，然后选中内容管理-->上传文件（这里先上传ipa文件）：

![上传文件.png](http://upload-images.jianshu.io/upload_images/1376067-97f55b29c96e1248.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7：上传ipa文件成功之后查看文件的外链接，如图：（你看到的链接可能并不是https的而是http的，这里后面再说明)

![外链接.png](http://upload-images.jianshu.io/upload_images/1376067-3c5f566747757dc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

8:复制ipa文件的外链，接下来就是要上传plist文件了，首先创建plist文件：

![plist文件内容.png](http://upload-images.jianshu.io/upload_images/1376067-5bfe42bb1f36c8db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在plist文件的url中填写你复制的ipa的外链。其他的只需填写bundle-identifier为你的项目的id即可，title中填写你app的名称，其他的可不变。然后保存plist文件。

9：接下来就是上传plist文件到七牛了，同ipa文件的上传一样，上传成功后点击外链接即可查看外链：
![plist.png](http://upload-images.jianshu.io/upload_images/1376067-688cc663867c7d9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

10：接下来是最重要的一步，因为你会发现你上传的plist文件的外链接并不是https的，而企业版分发的plist下载链接必须是https。所以别急.先在查看内容中保存默认域名，然后选中你创建的空间名称，你会看到页面上有个“融合CDN加速域名”的title，点击添加HTTPS域名：

![点击保存域名.png](http://upload-images.jianshu.io/upload_images/1376067-19716f4069399085.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![https域名.png](http://upload-images.jianshu.io/upload_images/1376067-2b5be6d164e1219c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好了，此时你再去内容管理中，点击你之前上传的plist文件查看它的外链接，你会发现它已经是https的了。
ok，https链接的plist文件已生成，复制链接，在代码中：
```
#warning 测试代码
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        UIApplication *application = [UIApplication sharedApplication];
        [application openURL:[NSURL URLWithString:[NSString stringWithFormat:@"itms-services://?action=download-manifest&url=https://你的plist外链接地址.plist"]]];
        exit(0);
    });
```
就可以升级了。

有时你会发现，在你要升级版本的时候，你虽然更换了ipa文件，但是plist文件地址所对应的url的升级包并没有变，或者你连plist文件中的url也更新重新上传，但是你再次打开plist的外链你会发现plist的内容并没有修改。为此我忙活了半天不知道为什么。后来才知道这是由于七牛缓存的原因，所以你每次要发版的时候，在你上传完ipa文件的时候，最后能去刷新一下plist文件和ipa文件，在个人看板中：

![个人中心.png](http://upload-images.jianshu.io/upload_images/1376067-7dcc8f44efc0ed89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


有个刷新预取得功能，复制ipa的链接或plist的外链到那里即可刷新（刷新是有次数限制的）：

![刷新预取.png](http://upload-images.jianshu.io/upload_images/1376067-8af27c4f8b3a6146.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


恩，刷新完时候你就可以愉快的升级了。（缓存很啃爹，请慢慢摸索）。
最后奉上plist文件的源码：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>ipa文件的外链，这个外链可以不是https的，也就是说http即可</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string></string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string></string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>你的bundleID</string>
				<key>bundle-version</key>
				<string>1.2.0</string>
				<key>kind</key>
				<string>software</string>
				<key>subtitle</key>
				<string>App</string>
				<key>title</key>
				<string>app名称</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

ok，以上总结有什么不对或瞎扯的地方欢迎在下方留言中指出。

ps：无demo不文章，抱歉，真没有demo。