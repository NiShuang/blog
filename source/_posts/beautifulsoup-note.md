---
title: Beautiful Soup 文档搜索方法(find_all find)中 text 参数的局限与解决方法

date: 2018-06-28 20:34:45
tags:
 - python 
 - 爬虫
 - HTML
categories: 爬虫笔记
---

## find_all方法介绍

`find_all( name , attrs , recursive , text , **kwargs )`

find_all() 方法搜索当前tag的所有tag子节点，并判断是否符合过滤器的条件。具体请看官方文档

> [Beautiful Soup 4.2.0 中文文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#find-all)

其中，对于text参数的介绍如下：

通过 text 参数可以搜搜文档中的字符串内容和tag。与 name 参数的可选值一样， text 参数接受 字符串 、 正则表达式 、 列表、 True 。 看例子:

```
soup.find_all(text="Elsie")
# [u'Elsie']

soup.find_all(text=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(text=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

soup.find_all("a", text="Elsie")
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]
```

<!-- more --> 

> 注意：如果使用 find_all 方法时同时传入了 text 参数 和 name 参数 。Beautiful Soup会搜索指定name的tag,并且这个tag的 **tag.string** 属性包含text参数的内容。结果中不会包含字符串本身。

## text 参数的局限

上面提到，text参数相当于搜索 tag 的 **tag.string** ， 而 tag.string 的规则如下：

> 如果tag只有一个 NavigableString 类型子节点,那么这个tag可以使用 .string 得到子节点

> 如果一个tag仅有一个子节点,那么这个tag也可以使用 .string 方法,输出结果与当前唯一子节点的 .string 结果相同

> 如果tag包含了多个子节点,tag就无法确定 .string 方法应该调用哪个子节点的内容, .string 的输出结果是 None 

所以当某个tag有多个子tag时，我们是无法通过text参数搜索到该 tag 的。比如下面这种情况：

```
<a>
	abc
	<div clss='no_print'>
	</div>
</a>
```
我们无法通过 `soup.find('a', text = 'abc')` 来搜索该 a 标签。


## 解决方法

这里我想到了一种方法，就是先把 a 标签中字符串之外的子元素删除：

``` python
[s.extract() for s in soup.find_all(name='div', class_='no_print')]
```
使要搜索的tag变成如下形式：

```
<a>
	abc
</a>
```
这样就可以通过 `soup.find('a', text = 'abc')` 来搜索该 a 标签。


另外，除了标签中带有别的标签，还会有**换行符**和**注释**等等，这些的存在都会导致该标签无法通过text参数来搜索到：

```
<a>
	abc
	<br>
	def
</a>

<a>
	abc
	<br>
	def
	<!--这是一段注释。注释不会在浏览器中显示。-->
</a>
```

换行符建议在 bs解析html文本之前，用`replace()`方法去掉：

```
html = html.replace('<br>', '').replace('<br/>', '')

```
因为bs对换行符的解析会有一点点迷。

而注释的删除比较特别：

```
from bs4 import BeautifulSoup, Comment
for comment in soup(text=lambda text: isinstance(text, Comment)):

```


## 参考资料

 - [Beautiful Soup 4.2.0 中文文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)

---

> 文章标题：[Beautiful Soup 文档搜索方法(find_all find)中 text 参数的局限与解决方法](http://www.cielni.com/2018/06/28/beautifulsoup-note/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2018/06/28/beautifulsoup-note/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！

	

