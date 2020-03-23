---
title: 动态改变checkbox的选中状态
date: 2017-06-12 17:59:34
tags:
 - js
categories: 前端笔记
---
### 问题

在写前端的时候，有一个需求是要用 js 代码动态改变复选框 checkbox 的选中状态。我的思路是通过改变checkbox元素的checked属性来实现页面上复选框是否选中的切换。

一开始的方法是通过jquery的attr()和removeAttr()方法来完成对元素checked属性的添加与修改，代码如下：

``` javascript
leader = $('#leader_modify');
            if(is_leader == '1'){
              leader.attr('checked', 'checked');
            }else{
              leader.removeAttr('checked')
            }
```

这样子写是基本可以完成checkbox的状态切换，但会有一个bug：切换在一开始是正常的，但当我点击了一次复选框以后，之前的状态切换就不起作用了（。。这不是坑爹吗！？）
<!-- more --> 
### 原因

经过一番漫无目的的搜索资料，我仔细研究了下 jquery的 [.attr()
](http://www.jquery123.com/attr/) 方法，attr() 可以获取匹配的元素集合中的第一个元素的属性的值，或者设置每一个匹配元素的一个或多个属性。而这个属性的英文为 Attribute，它有别于Property。 

- property是DOM中的属性，是JavaScript里的对象
- attribute是HTML标签上的特性，它的值只能够是字符串

boolean attributes，比如：checked，仅被设置成默认值或初始值。在一个checkbox的元素中，checked attributes在页面加载的时候就被设置，而不管checkbox元素是否被选中。

properties就是浏览器用来记录当前值的东西。正常情况下，properties反映它们相应的attributes(如果存在的话)。但这并不是boolean attriubutes的情况。当用户点击一个checkbox元素或选中一个select元素的一个option时，boolean properties保持最新。但相应的boolean attributes是不一样的，正如上面所述，它们仅被浏览器用来保存初始值。

### 解决方法

由此可见，通过改变 checked 这个 attribute 来实现checkbox 状态的动态改变是不可行的，应该通过设置 checkbox 的 property 属性 来实现。jquery 提供了[.prop()](http://www.jquery123.com/prop/) 方法。

``` javascript
leader = $('#leader_modify');
            if(is_leader == '1'){
                leader.prop('checked', true)
            }else{
                leader.prop('checked', false);
            }
```

### 总结

> *attributes 和 properties*之间的差异在特定情况下是很重要。**jQuery 1.6之前** ，.attr()方法在取某些 attribute 的值时，会返回 property 的值，这就导致了结果的不一致。**从 jQuery 1.6 开始**， .prop()方法返回 property 的值,而 .attr()  方法返回 attributes 的值。

通过prop()来获取输入框里面的值永远都是和它里面的值同步的，而通过attr()老获取输入框里面的值一直都是在html标签里面设置的值。

根据官方的建议：**具有 true 和 false 两个属性的属性，如 checked, selected 或者 disabled 使用prop()，其他的使用 attr()。**

### 参考资料

- [Web前端之复选框选中属性](http://www.cnblogs.com/yuanzm/p/4125032.html) 
- [checkbox选中属性---坑到死](http://blog.csdn.net/brucecheng22/article/details/50408199)
- [DOM 中 Property 和 Attribute 的区别](http://www.cnblogs.com/elcarim5efil/p/4698980.html) 