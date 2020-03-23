---
title: mysql 中 null 的查询问题
comments: true
date: 2019-02-05 18:43:25
tags: 
  - 数据库
  - MySQL
categories: 后端笔记
---
### <> 与 != 查询不到 null
```
// 查询结果中不包括 null
SELECT id FROM table where value <> 1 
SELECT id FROM table where value != 1 

// 查询结果中包括 null
SELECT id FROM table where value is not 1 

```
### 查询 null 需要用 is 和 is not
```
// 查询影响不了 null
SELECT id FROM table where value != null 

// 查询影响 null
SELECT id FROM table where value is not null
```