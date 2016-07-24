---
layout: post
title: "Xcode路径改变导致CocoaPods报错的解决办法"
date: 2016-07-22 11:12:09 +0800
comments: true
categories: 
---
这几天心血来潮，更新了下Mac系统，然后装了下Xcode8装逼，接下来就懵逼了。
<!--more-->
由于之前的Xcode是7.1的一直没升级，本想体验体验Xcode8的，所以装完Xcode8之后Xcode7.1也没有删掉，一直两个Xcode一起用着，用了几天之后觉得Xcode8挺稳定的，于是就删了Xcode7，故事由此开始了。

删了Xcode7的几天后，发现Xcode8 bug还是蛮多的，不是很稳定，并且大家都不怎么推荐这么早就使用Xcode8，于是忍痛割爱（要知道这可是我下了好几个小时才下下来的啊）把Xcode8删了准备用回Xcode7.1。于是我就从废纸篓里恢复了几天前删除了的Xcode7.1.庆幸竟然还能用（暗暗窃喜).

今天由于要新增一个第三方，所以我很熟练的vim Podfile
巴拉巴拉巴拉然后再 pod install，接下来就懵逼了。。。。

报如下错误:

```
Setting up CocoaPods master repo
  $ /usr/bin/git remote set-url origin https://github.com/CocoaPods/Specs.git
  xcrun: error: active developer path ("/Applications/Xcode.app/Contents/Developer") does not exist, use `xcode-select --switch path/to/Xcode.app` to specify the Xcode that you wish to use for command line developer tools (or see `man xcode-select`)
  [!] Failed: /usr/bin/git remote set-url origin https://github.com/CocoaPods/Specs.git
  $ /usr/bin/git checkout master
  xcrun: error: active developer path ("/Applications/Xcode.app/Contents/Developer") does not exist, use `xcode-select --switch path/to/Xcode.app` to specify the Xcode that you wish to use for command line developer tools (or see `man xcode-select`)
  [!] Failed: /usr/bin/git checkout master
[!] Unable to add a source with url `https://github.com/CocoaPods/Specs.git` named `master-1`.
(The `master` repo is not a git repo.)
You can try adding it manually in `~/.cocoapods/repos` or via `pod repo add`.
```

试了几次都不行，又换pod update还是不行，于是我开始怀疑是不是COcoaPods炸了，还是我升级了系统之后炸了。。。。想想不科学，应该是XCode的问题，仔细阅读错误（原谅我英文差），貌似看懂了那么一点，可能是我删了XCode又恢复导致的。就这么一百度果然还真是的。
一句命令搞定：sudo xcode-select --switch /Applications/Xcode.app（后面的地址直接打开程序把Xcode往这里拖即可)。完事后再pod install。。biubiubiu出现了熟悉的“Analyzing dependencies”字样，最后提示“Pod installation complete! There are 15 dependencies from the Podfile and 16 total pods installed.”。ok成功了。

以上记录我遇到的坑爹错误，给初次遇到这样问题的童鞋一点思路，不至于搓手不急（当时CocoaPods不能用我心中一万个草泥马崩腾啊）