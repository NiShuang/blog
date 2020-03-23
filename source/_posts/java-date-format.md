---
title: Java 中 YYYY-MM-dd 在跨年时的致命问题
comments: true
date: 2020-01-10 20:57:58
tags: java
category: 后端笔记 
---

最近在[V站](https://v2ex.com/t/633650)和[知乎](https://zhuanlan.zhihu.com/p/100648038)都看到一个讨论很有意思，故做一下笔记。

## 问题描述

在跨年期间，如果在日期格式化的时候使用 YYYY 来格式化年份，则可能会出现下图所示的bug：

<!-- more --> 

![BUG](https://pic3.zhimg.com/80/v2-d503911afd31fedacd98605e624426de_hd.jpg)

## 根本原因

YYYY 在官方文档中的解释是 week-based-year，**表示当天所在的周属于的年份**，一周从周日开始，周六结束，只要本周跨年，那么这周就算入下一年。所以2019年12月31日那天在这种表述方式下就已经是 2020 年了。而当使用 yyyy 或者 uuuu 的时候，就还是 2019 年。

## 总结
```
u year
y year-of-era
Y week-based-year，表示当天所在的周属于的年份
```
u 与 y 在公元后的年份表示没有区别，在公元前的年份表示有正负号的差别。所以建议平时时期格式化的时候使用 yyyy-MM-dd 或者 uuuu-MM-dd。

## 参考资料

 - [你今天因为 YYYY-MM-dd 被提 BUG 了吗
](https://v2ex.com/t/633650)
 - [昨天你用的 YYYY-MM-dd 被同事锤了吗？](https://zhuanlan.zhihu.com/p/100648038)
 - [Class DateTimeFormatter 官方文档](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#patterns)
-------

 > 文章标题：[Java 中 YYYY-MM-dd 在跨年时的致命问题](http://www.cielni.com/2020/01/10/java-date-format/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2020/01/10/java-date-format/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！