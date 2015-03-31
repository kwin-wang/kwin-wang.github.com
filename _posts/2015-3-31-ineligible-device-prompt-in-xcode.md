---
layout: post
title: "xcode不能真机调试，提示:noeligible device"
description: ""
category: "编程"
tags: [iOS]
---
{% include JB/setup %}

##问题情境1: Xcode不能真机调试

在我的MacBook Air上部署好iOS编程环境后，在打开一个iOS项目时，其可以在模拟器上调试，但是当把我的iPad设备插入到电脑后，Xcode虽然能识别我的设备，但是点击`Device`标签后下拉的设备选择菜单中却出现: `noelegible device`，并且不能选择我的iPad作为调试设备，然后就没有然后了...


##解决方法

最后经调研发现，我的Project的`Deployment Target`设置的是`7.0`，而我的iPad的iOS系统是6.2的版本，由于设置的目标系统版本不对，才导致Xcode不能选择真机调试。

将`Deplyment Target`的版本设置成`6.0`后，就能真机调试了



##问题情境2: 怎样将一个Mac上得开发者证书导入到另一个Mac中

不同的Xcode的版本可能不一样，拿Xcode6.2为例：

1. 打开Xcode后，选择:Xcode->Preferences->Accounts
2. 点击下方的设置图标，选择`Exports Accounts`(注: 是图标),将文件保存到电脑上
3. 将到处的证书文件在另一个mac电脑点击打开即可安装



