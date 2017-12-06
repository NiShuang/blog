---
title: kekbab-case （短横线隔开命名法）
date: 2017-07-08 00:17:33
tags:
 - Vue
categories: 前端笔记
---

在Vue.js中，由于 HTML 特性不区分大小写。名字形式为 camelCase 的 prop 用作特性时，需要转为 kebab-case（短横线隔开）。
即 myMessage 转为 my-message 。

官网例子：

``` javascript
Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```
``` html
<!-- kebab-case in HTML -->
<child my-message="hello!"></child>
```