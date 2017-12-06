---
title: 正则表达式的贪婪匹配与非贪婪匹配
date: 2017-09-07 01:30:54
tags: 
 - python 
 - 爬虫 
 - 实习
categories: 爬虫笔记
---

### 情景

之前写过一个简单的爬虫，每天获取公司[insgtagram主页](https://www.instagram.com/insta360official/)的粉丝数用来进行粉丝趋势的展示。代码很简单就是通过获取主页源代码后用正则表达式匹配其中的一串json数据，再用python的json解析库进行解析，从中获取粉丝数。

然而昨天这个爬虫报错了，我重新看了一下ins主页的网页源代码，发现其中增加了一段内容，这段内容正好被我匹配进去了。~~经过思考~~显而易见，这是贪婪匹配的问题。
<!-- more --> 

### 贪婪匹配与非贪婪匹配

现在这些术语听起来都很吓人，其实这是正则匹配的两种模式，很容易解释：

- 贪婪匹配：尽可能匹配最长的字符串
- 飞贪婪匹配： 尽可能匹配最短的字符串

举个例子：
`aa<div>test1</div>bb<div>test2</div>cc `

如果想要匹配一个完整的div，贪婪模式的结果为：
`<div>test1</div>bb<div>test2</div> `

非贪婪模式的结果为：
`<div>test1</div>`

可以发现贪婪模式会匹配尽可能长的字符串，而非贪婪模式在第一次匹配成功后就会停止匹配。

### 如何区分两种模式

默认情况下匹配都是贪婪模式，如果要改成非贪婪模式，只需要量词后面加上一个问号?。

常用的量词有：
- `{m,n}`
- `{m,}`
- `?`
- `*`
- `+`

这些默认都是贪婪模式，若改成非贪婪模式，只需这样：
- `{m,n}?`
- `{m,}?`
- `??`
- `*?`
- `+?`

针对上面那个div的例子，贪婪模式的匹配表达式为：
`<div>.*</div> `

非贪婪模式的匹配表达式为：
`<div>.*?</div> `

### 总结

贪婪模式就是匹配最长的字符串，非贪婪模式就是匹配最短字符串。

默认情况下匹配都是贪婪模式，如果要改成非贪婪模式，只需要量词后面加上一个问号?。

使用贪婪模式还是非贪婪模式，这主要取决于我们的需求。但有一点，非贪婪模式的性能一定是高于贪婪模式的。

最后，附上我的爬虫代码：

``` python 
# -*- coding: UTF-8 -*-
import json
import requests
import re

def get_by_request():
    username = 'insta360official'
    url = 'https://www.instagram.com/' + username + '/'
    response = requests.get(url=url, verify=False)
    page = response.text
    pattern = re.compile("window._sharedData = (.*?);</script>", re.S)
    items = re.findall(pattern, page)
    jsonData = json.loads(items[0])
    count = jsonData['entry_data']['ProfilePage'][0]['user']['followed_by']['count']
    print count
    return count

if __name__ == "__main__":
    get_by_request()

```

---

> 文章标题：[正则表达式的贪婪匹配与非贪婪匹配](http://www.cielni.com/2017/09/07/greedy-matching/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2017/09/07/greedy-matching/
> 有问题或建议欢迎在[我的博客](http://www.cielni.com)讨论，转载或引用希望标明出处，感激不尽！