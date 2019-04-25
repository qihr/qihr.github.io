---
title: UnityiOSSDK接入
tags: [iOS, Unity, Lua,SDK]
categories: Program
---
SDK接入流程

<!-- more -->

# 一 导出Xcode工程

## 1.1修改cs文件配置plist



## 1.2修改iOS配置表



## 1.3修改打包列表



## 1.4修改资源

（替换图标  添加UnityAppController.mm ... 添加SDK(.a .framework bundle ...) ）

## 1.5导出Xcode工程



# 二 接入SDK

## 2.1修改项目配置

（证书 内购 ）

## 2.2根据文档接入SDK





# 三 Lua中配置SDk

## 3.1配置渠道信息表



## 3.2添加对应SDK的Lua脚本



## 3.3添加新引入的SDK平台 （PlatformSDK.lua）



# 四 出包

## 4.1版本号

修改版本号文件夹（测试稳定后更为正式版本号）

## 4.2 热更测试

通知运维 

## 4.3出包