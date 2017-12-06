---
title: Vue 实例生命周期总结
date: 2017-06-05 18:00:21
tags:
 - Vue
categories: 前端笔记
---

## 问题

由于Vue项目中需要前端导出表格的功能，需要用到第三方组件tableexport。使用该组件只需通过该语句TableExport(document.getElementsByTagName("table"))生成一个TableExport对象。然而我却遇到一个bug，迟迟得不到预期，花了一整个白天没有解决。

睡觉之前我打印了document.getElementsByTagName("table")发现是undefined，这时候我意识到我生成TableExport对象的时机错误了，此时该table元素还没有插入文档中。于是我把这行代码从created钩子函数中移到了ready钩子函数中（vue 1.0），bug终于解决了。

<!-- more --> 

之前的代码是这样子的：
``` javascript
  created () {
    this.startTime = '2016-06-01'
    this.endTime = moment().format('YYYY-MM-DD')
    this.updateColor()
    this.serial_numbers = ''
    this.product = 'nano'
    TableExport(document.getElementsByTagName("table"))
  },

  ready () {
    
  }
```

修改后的代码是这样子的：
``` javascript
  created () {
    this.startTime = '2016-06-01'
    this.endTime = moment().format('YYYY-MM-DD')
    this.updateColor()
    this.serial_numbers = ''
    this.product = 'nano'
  },

  ready () {
    TableExport(document.getElementsByTagName("table"))
  }
```
## 原因

这是由于在created钩子执行时，Vue实例虽已经创建完成，但模板还未被编译，元素还未插入DOM文档，所以document.getElementsByTagName("table")返回的是undefined。而在ready钩子执行时，模板已被编译，元素已插入DOM文档。

## 总结

查阅了官网并参考网上其他资料后，我总结出了如下了的Vue 实例生命周期表

| Vue 1.0  | Vue 2.0   |  描述  |
| --------   | -----  | ----  |
| init     |beforeCreate| 实例初始化之后，配置之前|
| created  |created| 实例已经创建完成之后，已完成配置，挂载之前。|
| beforeCompile     |-    | 模板编译开始前 |
| -     |beforeMount    |  挂载之前 |
| compiled     |-    |模板编译，但是不担保el已插入文档）|
| ready     |mounted| 模板编译（el第一次插入文档之后）/挂载之后|
| -     |beforeUpdate |数据更新前|
| -     |updated| 数据更新后|
| -     |activated | keep-alive 组件激活时|
| -     |deactivated |keep-alive 组件停用时|
| attached    |- |el插入DOM时|
| detached     |-  |el从DOM中删除时|
| beforeDestory     |beforeDestory  | 实例销毁之前  |
| destoryed     |destoryed  | Vue 实例销毁后，Vue 实例指示的所有东西都会解绑，所有的子实例也会被销毁 |