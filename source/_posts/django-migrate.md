---
title: 解决Django中manage.py migrate无效的问题
date: 2017-05-30 03:24:59
tags: 
 - python
 - Django
categories: python笔记
---


## 介绍

Python的Django框架中有一个对象关系映射器，我们可以在其中以Python代码描述数据库布局，比如像这样：
``` python
from django.db import models

class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):              # __unicode__ on Python 2
        return self.full_name

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter)

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```
<!-- more -->
在编写好我们的数据模型models.py后，我们会依次运行如下两条命令来自动创建数据库表：

1. python manage.py makemigrations
2. python manage.py migrate

第一条命令会并记录下你所有的关于modes.py的改动，并在migrations包中生成类似0001_initial.py的文件，但是这个改动还没有作用到数据库。

第二条命令就是比照migrations中的文件，将改动作用到数据库文件。

## 问题描述

在开发过程中，由于种种原因，migrations包中的文件往往不能和实际的模型变化情况保持同步，这会导致在python manage.py migrate的时候产生冲突，无法作用到数据库。

## 原因分析

仔细分析可知，migrations中的每一个脚本文件记录了每一次models.py的改动，这些记录串联起来就是数据库从初始化到现在的变化过程。系统就是根据这些脚本来确定数据库目前的模型状态。如果之前的记录脚本缺失，那么系统认定的数据库状态就会和实际上models.py的状态有所偏差。

举个例子，我们对models.py中的User表进行添加字段的操作，而系统由于缺失了某一个记录脚本，认为不存在User表，在migrate时就会报表缺失的错误信息。

## 解决方法

我不建议网上说的删掉全部migrations文件重新建表的办法，因为如果表中存在重要数据的话重新建表示不可取的。

Django会记录系统migrate操作执行到了哪一个记录脚本，这一个脚本以及之前的所有脚本决定了系统认为的目前数据库的状态。所以我们只要对系统最近一次成功migrate时所执行的脚本进行修改，添加数据库的操作信息，使得脚本文件的记录和目前models的状态同步，告诉系统这个User表我已经创建过了。然后再对models修改，进行makemigrations和migrate操作，就不会报表缺失的错误了。

``` python
migrations.CreateModel(
    name='User',
    fields=[
        ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
        ('name', models.CharField()),
    ],
),
```