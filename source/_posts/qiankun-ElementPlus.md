---
title: qiankun+ElementPlus 弹出类组件失效
date: 2025-04-07 15:59:22
tags: [ElementPlus, qiankun, micro-frontends]
categories: [frontend]
---

#### 问题描述：在子应用中使用的ElMessage、ElSelect的下拉框失效

需求新增基座，子应用为老项目做微前端改造，想着不希望基座对子应用产生样式影响，开始采用`strictStyleIsolation`严格隔离。
项目改造完成启动后发现子应用中message与select选项无法弹出。随后想到ElementPlus对`弹出类组件`设计了统一容器，message与下拉列表均为`动态挂载`。
Function触发后会被动态挂载到`body`下的`el-popper-container-xxxx（）`内，严格模式内会导致dom动态加载至`#shadow-root`外从而无法在子应用内展示。
所以最终的目标是将el-popper-container挂载至`子应用`容器内。

![](shadow-root.png)

#### 个人解决方案

##### 隔离方式采用experimentalStyleIsolation

在严格隔离状态下尝试获取沙箱内dom失败，为了解决动态加载时找不到子应用容器问题，遂转换成`experimentalStyleIsolation`隔离方案。

在`实验性隔离`方案下，el-popper-container仍挂载至父应用body下，但好消息是可以取到子应用dom，通过`重写append方法`在动态挂载时更改挂载节点来解决问题。

![](get-id.png)

##### 子应用重写append方法

在子应用render时如果是qiankun环境会接收到父应用传递的用于子应用挂载的容器id，那么可以在`子应用`render中添加一个执行条件。

```bash
const render = (props = {}) => {
  if(!!props.container) {
    redirectPopup(props)
  }
  
  // do something else
}} 
```

下来就是要在redirectProps这个函数中实现将错误挂载在父应用body下的popper与message组件正确挂载至子应用容器下。

```bash
const redirectPopup = (props) => {
    // 需要正确挂载到子应用的弹窗className白名单。
    const whiteList = ['el-popper-container']

    // 保存原有document.body.appendChild方法
    const originFn = document.body.appendChild.bind(document.body)

    // 重写appendChild方法
    document.body.appendChild = (dom) => {
        // 根据标记，来区分是否用新的挂载方式
        whiteList.forEach((x) => {
            if (dom.id.includes(x)) {
                // 有弹出框的时候，挂载的元素挂载到子应用上，而不是主应用的body上
                // #traveling-wave-container为子应用body下容器
                props.container.querySelector('#traveling-wave-container').appendChild(dom)
            } else {
                originFn(dom)
            }
        })
        // 是否el-message
        if (dom.classList.value && dom.classList.value.includes('el-message')) {
            props.container.querySelector('#traveling-wave-container').appendChild(dom)
        }
    }
}
```

此时再触发可看到`el-popper-container-xxxx`已成功append至子组件容器内，并且popper与message等弹出层已正常。

![](img3.png)

***

在沙盒模式中应该也存在强制挂载的方法 等有时间了看看再更


