---
layout: post
title: "iOS上传图片文件到七牛遇到的那些坑"
date: 2016-03-12 23:35:37 +0800
comments: true
categories: 
---
最近公司的项目涉及到图片的保存，基于大量图片放在本地服务器会给公司增加不小的压力，所以选择了云存储，对比之后选择了七牛。

  七牛的官方文档实在是太过于简单，下载SDK之后，根据官方 给的Demo，成功上传了图片。只是token的生成过程有些波折。

由于七牛官方不提倡在客户端生成Token，所以没有给出相应的iOS端生成Token的代码。多方查找资料之后自己写了一个本地生成Token的方法如下：

有了token之后就引入SDK，如下操作即可实现上传：

一开始很纳闷，从放回信息中没有找到我们上传成功之后需要的图片的外链接地址，查阅官方文档之后才明白外链接地址需要我们自己拼接。首先，成功之后返回的resp是：

resp {

              hash = "FhweZwfJipE4P0K6Mm_QbC6P0dxW";

              key = zx12;

}

如果失败resp为nil，所以图片的外链接地址就是你的 域名/key(返回的key)

此demo下载地址：https://github.com/voidxin/UploadImageToQiniu