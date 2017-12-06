---
title: 如何爬取 Google Trends 数据
date: 2017-05-28 23:27:02
tags: 
 - python 
 - 爬虫 
 - 实习
categories: 爬虫笔记
---
## Goole Trends介绍

### 功能介绍

[谷歌趋势](https://trends.google.com/trends/) (Google Trends)是Google推出的一款基于搜索日志分析的应用产品，它通过分析Google全球数以十亿计的搜索结果，告诉用户某一搜索关键词各个时期下在Google被搜索的频率和相关统计数据。（国内访问Google Trends需要翻墙）
<!-- more --> 
### 指数意义

Google Trends在展示趋势的时候会隐藏真实的搜索量，它的逻辑是这样的，在一段时间内，选取这个词条最热的那一天当做100，其他天数的指数是相对于100这天的相对值。如果有很多个词条的话，就选择所有词条中最热那天的数据作为100，其他的词条和其他的日期都是这一天的相对值。也就是说，所有Google Trends给出的热度数据中，最高值就是100，其他值都是相对于最高值进行等比计算的。

![例子](http://ww1.sinaimg.cn/large/006HJ39wgy1fg1imshjxyj30w90f2dge.jpg)
上图是一个热度随时间变化的趋势的例子，可以看到，图中的最大值就是100，也就是时间段内的最高点。

## 爬取方法

Google Trends的爬虫流程图如下所示：

![爬虫思路](http://ww3.sinaimg.cn/large/006HJ39wgy1fg1j952jphj30jh056gll.jpg)

我研究Google Trends爬虫的心路历程就不细讲了，当时为了找关键词的token值花了我两天的时间，几乎都想放弃了。后来跟旁边的同事讲了下思路和问题，他听后很明确地告诉我，token就在network的请求里，你一个一个找肯定能找到。果然后面我很快就找到了。

下面我详细讲解最重要的前两步操作。

### 获取关键词的token值

获取关键词token值的接口为： https://trends.google.com/trends/api/explore
主要难点是参数的拼接，希望大家对照着浏览器network中的参数一个一个仔细检查自己的参数格式是否有误。
以获取关键词insta360的最近30天热度走势为例，请求token值需要携带的参数为：

![](http://ww1.sinaimg.cn/large/006HJ39wgy1fg1jwbfqnxj30hy02edfo.jpg)

请求中的req参数是一个json格式的字符串，如果你的请求不通过，很有可能是这个参数的格式或者数值有问题吗，一定要仔细检查。

如果参数正确，将会得到如下的返回结果：

![](http://ww3.sinaimg.cn/large/006HJ39wgy1fg1k5fv5xqj30rk043dfs.jpg)

这是谷歌一个很奇怪的地方，他不直接返回json数据，而是在最开头加了五个蜜汁符号，我们只要截取掉前五个字符就可以了。然后通过json库把惊悚字符串转换成python的dict数据格式，就可以得到token值，它的索引位置为['widgets'][0]['token']。这里给大家推荐[Json.cn](http://json.cn/)，可以在线解析json字符串，方便我们在其中找到我们想要的信息。

最后注意的是，我们需要用到python的requests库来发送请求，因为该请求会自动重定向导致我们获取不到数据，需要设置allow_redirects=False来拒绝重定向。

附上代码：
``` python
def get_token(key):
    headers = {}
    headers['Host'] = 'trends.google.com'
    headers['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:49.0) Gecko/20100101 Firefox/49.0'
    headers['Referfer'] = 'https://trends.google.com/trends/explore?date=today%201-m&q=' + urllib.quote(key)
    headers['x-client-data'] = 'CIu2yQEIpLbJAQjBtskBCPqcygEIqZ3KAQ=='
    req = {}
    req['category'] = 0
    req['property'] = ''
    req['comparisonItem'] = [{"geo": "","keyword":  urllib.quote(key).replace(' ', '+'),"time":"today+1-m"}]
    value = {}
    value['hl'] = 'zh-CN'
    value['tz'] = '-480'
    value['req'] = str(req).replace(' ','')
    url = 'https://trends.google.com/trends/api/explore?'
    for index in value:
        url = url + index + '=' + value[index] + '&'
    # 后面两个参数很重要
    results = requests.get(url, headers=headers, verify=False, allow_redirects=False)
    page = results.content
    jsonData = page[5:]
    data = json.loads(jsonData, encoding="utf-8")
    token = data['widgets'][0]['token']
    return token
```

### 携带token值请求数据

这一步和上一步很像，只是变了请求接口和请求的参数，而且该请求返回的数据格式和上面很像，同样需要截掉开头五个符号。

获取关键字热度趋势数据的接口为： https://trends.google.com/trends/api/widgetdata/multiline

该请求的参数为：

![](http://ww1.sinaimg.cn/large/006HJ39wgy1fg1kl7kek3j310802zaa2.jpg)

其中，token的值就是上面获取的token。在req参数中，time的值需要我们拼接。格式为*start_date*+*end_date*，end_date是当天日期，start_date是往前推三十天的日期。如果不知道怎么计算前三十天的时期，可以用我最原始的方法:
``` python
now = datetime.datetime.now()
month = (now.month + 10) % 12 + 1
year = now.year - month / 12
day = min(now.day, calendar.monthrange(year, month)[1])
before = now.replace(year=year, month=month, day=day)
start_date = before.strftime('%Y-%m-%d')
end_date = now.strftime('%Y-%m-%d')
```

附上代码：
``` python
def get_google_trend(key):
    headers = {}
    headers['Host'] = 'trends.google.com'
    headers['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:49.0) Gecko/20100101 Firefox/49.0'
    headers['Referfer'] = 'https://trends.google.com/trends/explore?date=today%201-m&q=' + urllib.quote(key)
    req = {}
    req['time'] = start_date + "+" + end_date
    req['resolution'] = "DAY"
    req['locale'] = "zh-CN"
    req['comparisonItem'] = [{"geo": {}, "complexKeywordsRestriction": {"keyword": [{"type": "BROAD", "value": urllib.quote(key).replace(' ','+')}]}}]
    req['requestOptions'] = {"property":"","backend":"IZG","category":0}
    value = {}
    value['hl'] = 'zh-CN'
    value['tz'] = '-480'
    value['req'] = str(req).replace(' ','')
    value['token'] = token[key]
    url = 'https://trends.google.com/trends/api/widgetdata/multiline?
    for index in value:
        url = url + index + '=' + value[index] + '&'
    results = requests.get(url, headers=headers, verify=False)
    page = results.content
    jsonData = page[5:]
    data = json.loads(jsonData, encoding="utf-8")
    items = data['default']['timelineData']
    result = []
    for item in items:
        timestamp = int(item['time'])
        time_temp = time.localtime(timestamp)
        date = time.strftime("%Y-%m-%d", time_temp)
        value = item['value'][0]
        temp = {'key': key, 'date': date, 'google_index': value}
        result.append(temp)
    return result
```

后面两步我就不讲了，第三步获取响应中的json数据已经在第一步中提过了。

最终得到的数据通过Json.cn解析后是下面这个样子

![结果](http://ww1.sinaimg.cn/large/006HJ39wgy1fg1naam29nj30fl0dv3yu.jpg)

Google Trends支持多个词条比较（最多五个），多个词条的对比数据爬取方式和单个词条类似，不同的是在参数部分需要把多个词条用逗号拼接，大家可以自行探索。

如有疑问或建议欢迎在评论区留言。