---
title: qiankun+ElementPlus 弹出类组件失效
date: 2025-06-07 15:59:22
tags: [qiankun, 微前端]
categories: [前端]
---

#### 问题描述：在子应用中使用的ElMessage、ElSelect的下拉框失效

需求新增基座，子应用为老项目做微前端改造，想着不希望基座对子应用产生样式影响，开始遂采用`strictStyleIsolation`严格隔离。
项目改造完成启动后发现子应用中message与select选项无法弹出。随后想到ElementPlus对`弹出类组件`设计了统一容器，message与下拉列表均为`动态挂载`。
Function触发后会被动态挂载到`body`下的`el-popper-container-xxxx（）`内，严格模式内会导致dom动态加载至`#shadow-root`外从而无法在子应用内展示。
所以最终的目标是将el-popper-container挂载至`子应用body`下。

![](shadow-root.png)

以下为个人解决方案

#### 1、采用experimentalStyleIsolation

在严格隔离状态下尝试获取沙箱内dom失败，为了解决动态加载时找不到子应用容器问题，遂转换成`experimentalStyleIsolation`隔离方案。

在`实验性隔离`方案下，el-popper-container仍挂载至父应用body下，但好消息是可以取到子应用dom。遂想到通过重写append来解决问题。


`layout`同层级的`el-popper-container-xxxx（）`内，在实验性隔离状态下，沙箱会导致dom在动态append时找不到子应用render时的容器。