---
title: MySQL 中 Group By 后如何选择记录
comments: true
date: 2019-08-17 19:19:15
tags: 
  - 数据库
  - MySQL
category: 后端笔记 
---
## 问题背景

有一张数据库表，记录了一些抖音视频每小时的播放量与点赞量，每个视频每小时都会产生一条记录。现在需要查出每个视频最新的一条记录，希望通过一条sql语句搞定。第一反应就是对 video_id 进行 Group By ，然后想办法取出每一组中 created_time 最新的那条数据。在最后加 Order By 显然是行不通的，因为 Order By 是对 Group By 的结果进行排序。

## 误区
关于这种问题，网上有很多错误的解决方法，思路是先通过一个子查询把数据按照 created_time 倒序排序，然后再进行 Group By，sql语句如下：

```
SELECT * FROM
(SELECT * FROM douyin_official_video WHERE ORDER BY created_time DESC) t
GROUP BY video_id
```

这个方法的成立需要一个前提，就是MySQL 在 Group By 后是按照当前数据排列顺序选择第一条记录的。

<!-- more --> 

然而我查阅了[MySQL 5.7 版本的官方文档](https://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html)

>If ONLY_FULL_GROUP_BY is disabled, a MySQL extension to the standard SQL use of GROUP BY permits the select list, HAVING condition, or ORDER BY list to refer to nonaggregated columns even if the columns are not functionally dependent on GROUP BY columns. **This causes MySQL to accept the preceding query. In this case, the server is free to choose any value from each group, so unless they are the same, the values chosen are nondeterministic, which is probably not what you want. Furthermore, the selection of values from each group cannot be influenced by adding an ORDER BY clause.**

这段话总结一下就是， 在 ONLY_FULL_GROUP_BY 这个配置关闭的情况下，MySQL 从 Group 中选择记录的方式是随意的，无论预先对源数据如何进行 Order By，都不会对选择有任何影响。

ONLY_FULL_GROUP_BY 这个配置决定了能否在 SELECT 后的字段中出现 Group By 后没有的字段。ONLY_FULL_GROUP_BY 为 disabled时，允许SELECT 后的字段中出现 Group By 后没有的字段；ONLY_FULL_GROUP_BY 为 enable 时， 只能 SELECT 在 Group By 后出现的字段。而在大多数情况下，为了减轻程序员编写sql语句的压力，ONLY_FULL_GROUP_BY 都会建议设为 disabled。

我的数据库 ONLY_FULL_GROUP_BY 是设为 disabled的，我跑了上述方法，事实证明这个方法确实是无效的，无论我预先如何排序，从 Group 中选择出来的记录都是 id 最小的那一条。但是我也不能说 Group By 以后就是选择 id 最小的那一条，因为文档中明确说明了是 free to choose，我们无法去做其他猜测。也许在有自增主键的情况下，是取主键最小的那一条，但是这个目前无法百分之百证实。

## 解决方案
因为在这个表中 created_time 最新也就意味着 id 最大，所以我变通一下，把问题简化为取每个视频中 id 最大的一条记录。这个问题可以通过聚合函数 MAX 先把每个视频的最大 id 查出来，然后在对这些 id 查询记录。sql语句如下：
```
SELECT * FROM douyin_official_video WHERE id IN 
(SELECT MAX(id) FROM douyin_official_video GROUP BY video_id) t
```

## 结论

在数据库 ONLY_FULL_GROUP_BY 是 disabled 的情况下：

```
SELECT * FROM table GROUP BY <字段 1>
```
随机选择一条

```
SELECT * FROM (SELECT * FROM table ORDER BY <字段 2>) GROUP BY <字段 1>
```
随机选择一条，而且子查询里面的 ORDER BY 会被优化掉。


## 参考资料

 - [mysql group by 后选择哪条记录](https://www.v2ex.com/t/379352)
 - [MySQL :: MySQL 5.7 Reference Manual :: 12.20.3 MySQL Handling of GROUP BY](https://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html)

-------

 > 文章标题：[MySQL 中 Group By 后如何选择记录](http://www.cielni.com/2019/08/17/mysql-group-by-select/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2019/08/17/mysql-group-by-select/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！

