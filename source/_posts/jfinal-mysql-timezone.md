---
title: 数据库查询的时区问题
date: 2018-07-14 17:55:14
tags:
 - java 
 - 实习
 - 数据库
categories: java笔记
---
## 需求

BI系统需要查询北京时间的某一天内公司所有设备激活记录，而数据库中记录的激活时间都是0时区的，系统需要不管在本地（GMT+8）还是服务器（GMT+0）上，都能准确查询导数据。

## 思路

首先我们需要得到北京时间某一天的起始时间和结束时间的时间戳：

```java
// 获取时间区间
Timestamp startTime = TimeKit.toChineseTimestamp(date); // 该方法是自己封装的，即把字符串的时间按照时区转化成时间戳
Timestamp endTime = new Timestamp(startTime.getTime() + 24 * 60 * 60 * 1000L);
```

然后使用该时间戳区间编写mysql语句，**值得注意的是，因为程序会自动把时间戳转化成字符串格式的时期去数据库进行比较，而这个转换是根据系统时区进行的，也就是说在本地和在服务器，转换出来的时间会差8个小时。**所以需要根据当前系统的时区，对时间戳作调整：

```java
// 因为mysql会将时间戳转换为字符串，所以减掉时区偏差
int offSet = TimeZone.getDefault().getRawOffset();
startTime.setTime(startTime.getTime() - offSet);
endTime.setTime(endTime.getTime() - offSet);
```
<!-- more --> 


## 完整代码


```java
protected void extractByDate(String date) {

// 获取时间区间
Timestamp startTime = TimeKit.toChineseTimestamp(date); // 该方法是自己封装的，即把字符串的时间按照时区转化成时间戳
Timestamp endTime = new Timestamp(startTime.getTime() + 24 * 60 * 60 * 1000L);

// 因为mysql会将时间戳转换为字符串，所以减掉时区偏差
int offSet = TimeZone.getDefault().getRawOffset();
startTime.setTime(startTime.getTime() - offSet);
endTime.setTime(endTime.getTime() - offSet);

// 从原表查找数据
String sql = "select device_type, location from device_activation where ? <= create_time and create_time < ?";
List<Record> rawRecords = primaryDbPro.find(sql, startTime, endTime);
```

---

> 文章标题：[数据库查询的时区问题](http://www.cielni.com/2018/07/14/jfinal-mysql-timezone/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2018/07/14/jfinal-mysql-timezone/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！