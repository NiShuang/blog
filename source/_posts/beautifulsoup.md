---
title: Beautiful Soup 采坑之旅
date: 2018-06-14 18:24:11
tags: 
 - python 
 - 爬虫
 - HTML
categories: 爬虫笔记
---

## Beautiful Soup入门

Beautiful Soup是一个Python库，用来解析html和xml结构的文档。具体关于Beautiful Soup的介绍与使用，可以参考以下资料：
> [Python爬虫利器二之Beautiful Soup的用法](https://cuiqingcai.com/1319.html)
> 
> [Beautiful Soup 4.2.0 中文文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)

下面是我在使用Beautiful Soup时遇到的小问题。
<!-- more --> 

## 解析器选择

官方对各个解析器的比较如下：

| 解析器        | 使用方法   |  优势  |  劣势  |
| --------   | -----  | ----  | ----  |
| Python标准库 | BeautifulSoup(markup, "html.parser")|  Python的内置标准库<br> 执行速度适中<br> 文档容错能力强 |Python 2.7.3 or 3.2.2前的版本中文档容错能力差|
| lxml HTML 解析器|BeautifulSoup(markup, "lxml")|  速度快<br>文档容错能力强| 需要安装C语言库 |
|lxml XML 解析器	|  BeautifulSoup(markup, ["lxml", "xml"])<br>BeautifulSoup(markup, "xml") | 速度快<br>唯一支持XML的解析器|需要安装C语言库|
| html5lib	| BeautifulSoup(markup, "html5lib")	 | 最好的容错性<br>以浏览器的方式解析文档<br>生成HTML5格式的文档|速度慢<br>不依赖外部扩展|

首先，如果是解析从网页上爬下来的HTML文档，请不要使用lxml XML 解析器，因为HTML解析器和XML解析器对于一文档的解析方式是不同的。**比如对于空标签&lt;b /&gt;,因为空标签&lt;b /&gt;不符合HTML标准,所以解析器把它解析成&lt;b&gt;&lt;/b&gt;,而使用XML解析时,空标签&lt;b/&gt;依然被保留。**

其次，在Python2.7.3之前的版本和Python3中3.2.2之前的版本中,标准库中内置的HTML解析方法不够稳定，所以我推荐使用lxml或者html5lib作为html文档的解析器,因为容错性比较好。

在HTML或XML文档格式正确的情况下，不同解析器的差别只是解析速度的差别。而在很多情况下，我们从网页上爬取的HTML文档会有格式不严谨的地方，那么在不同的解析器中返回的结果可能是不一样的，此时用户需要选择合适的解析器来满足自己的需求。

```python
# lxml解析
BeautifulSoup("<a></p>", "lxml")
# <html><body><a></a></body></html>						未补全
BeautifulSoup("<a><p>", "lxml")
# <html><body><a><p></p></a></body></html>				补全

# html5lib库解析
BeautifulSoup("<a></p>", "html5lib")
# <html><head></head><body><a><p></p></a></body></html>	补全
BeautifulSoup("<a><p>", "html5lib")
# <html><head></head><body><a><p></p></a></body></html>	补全

#Python内置库解析
BeautifulSoup("<a></p>", "html.parser")	
# <a></a>												未补全
BeautifulSoup("<a><p>", "html.parser")
# <a><p></p></a>										补全
```

从上面的例子可以看出，html5lib对于文档的容错性是最好的，它能补全大多数的标签。**而lxml和python内置解析器会忽略结束标签，补全开始标签。**

而对于部分没有结束标签的标签比如`<input/>`、`<img/>`等，在正常情况下，解析器都会正确解析，但如果是漏掉'/'的情况下，例如`<input><a></a>`:

```python
# lxml解析
BeautifulSoup("<input><a></a>", "lxml")
# <html><body><input/><a></a></body></html>					补全

# html5lib库解析
BeautifulSoup("<input><a></a>", "html5lib")
# <html><head></head><body><input/><a></a></body></html>	补全

#Python内置库解析
BeautifulSoup("<input><a></a>", "html.parser")
# <input><a></a></input>									未补错误
```

可见，Python内置库解析无法正确补全不需要结束标签的标签，比如`<input>`。


## find_all()的attrs参数

在find_all()方法中，如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索。如传入 href 参数,Beautiful Soup会搜索每个tag的”href”属性:

```python
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```
假如我们想用 class 过滤，不过 class 是 python 的关键词，这怎么办？加个下划线就可以

```python
soup.find_all("a", class_="sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

但有些tag属性在搜索不能使用,比如HTML5中的 `data-*` 属性，同时name由于已经是`find_all()`方法中的一个参数名（代表tag的名字），所以也不可通过tag中的name属性来搜索tag，但是可以通过 `find_all()` 方法的 attrs 参数定义一个字典参数来搜索包含特殊属性的tag，例如：

```python
data_soup.find_all(attrs={"data-foo": "value"})
# [<div data-foo="value">foo!</div>]
```


## .string 和 get_text()的区别

在Beautiful Soup，有两种获取标签内容的方法：.string属性 和 get_text()方法。

- .string 用来获取标签的内容 ,返回一个 NavigableString 对象。
 - 如果tag只有一个 NavigableString 类型子节点,那么这个tag可以使用 .string 得到子节点。

 - 如果一个tag仅有一个子节点,那么这个tag也可以使用 .string 方法,输出结果与当前唯一子节点的 .string 结果相同。

 - 如果tag包含了多个子节点,tag就无法确定 .string 方法应该调用哪个子节点的内容, .string 的输出结果是 None。

-  get_text() 用来获取标签中所有字符串包括子标签的内容，返回的是 unicode 类型的字符串

实际场景中我们一般使用 get_text 方法获取标签中的内容。





## .next_sibling 和 find_next_sibling()

在文档树中,使用 .next_sibling 和 .previous_sibling 属性来查询兄弟节点。实际文档中的tag的 .next_sibling 和 .previous_sibling 属性通常是字符串或空白。例如:

```
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>
```

如果以为第一个<a>标签的 .next_sibling 结果是第二个<a>标签,那就错了,真实结果是第一个<a>标签和第二个<a>标签之间的顿号和换行符:

```python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

link.next_sibling
# u',\n'

link.next_sibling.next_sibling
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
```

所以我建议使用 find_next_sibling() 方法来查询兄弟节点：

```python
link.find_next_sibling("a")
# <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>

link.find_next_siblings("a")
# [<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>,
# <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>]

```


## 换行符的问题

在HTML文档中经常会出现一些用来换行`<br>`标签，比如：

```
<div>
  some text <br>
  <span> some more text </span> <br>
  <span> and more text </span>
</div>
```

Beautiful Soup会将其自动补全为以下错误的形式：

```
<div>
  some text
  <br>
    <span> some more text </span>
    <br>
      <span> and more text </span>
    </br>
  </br>
</div>
```

因为`<br>`标签是为了展示的美观而出现的，而我们在解析文档时，这种标签的出现会影响我们解析的正确性（就如上面那个例子所示）。为了解决这个问题，我们需要使用extract()方法将文档中的`<br>`标签删掉

```python
soup = BeautifulSoup(text)
for linebreak in soup.find_all('br'):
	linebreak.extract()
```

这样最终的文档格式就变为：

```
<div>
  some text
    <span> some more text </span>
    <span> and more text </span>
</div>
```

## 大小写问题

因为HTML标签是大小写敏感的,所以3种解析器再出来文档时都将tag和属性转换成小写。例如文档中的 <TAG></TAG> 会被转换为 <tag></tag> 。如果想要保留tag的大写的话,那么应该将文档 解析成XML。


## 参考资料

 - [Beautiful Soup 4.2.0 中文文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)
 - [Python爬虫利器二之Beautiful Soup的用法](https://cuiqingcai.com/1319.html)
 - [python爬虫入门教程--HTML文本的解析库BeautifulSoup（四）](https://www.jb51.net/article/114663.htm)
 - [Beautifulsoup sibling structure with br tags](https://stackoverflow.com/questions/17639031/beautifulsoup-sibling-structure-with-br-tags)
 - [（学习笔记）Python BeautifulSoup4 取值部分](https://blog.csdn.net/zjiang1994/article/details/52679174)


---

> 文章标题：[Beautiful Soup 采坑之旅](http://www.cielni.com/2018/06/14/beautifulsoup/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2018/06/14/beautifulsoup/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！

	

