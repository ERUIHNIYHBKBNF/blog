---
title: '[爬虫笔记] 初探BeautifulSoup库'
date: 2021-01-18 09:52:35
tags: [python, 爬虫, web]
---

> Soup是汤啊..咱还以为是肥皂qwq

## 基本使用

beautifulsoup类， html文档， 标签树(字符串) 认为等价。

**bs4库将所有读入的html文件和字符串都转化为UTF-8编码**

```python
# pip install beautifulsoup4
from bs4 import BeautifulSoup # 注意大小写
soup = BeautifulSoup("<html>data</html>", 'html.parser') # 需要解析的html信息，解析器
soup = BeautifulSoup(open("D://demo.html"), 'html.parser')
```

| 解析器           | 使用方法      | 条件                 |
| ---------------- | ------------- | -------------------- |
| bs4的html解析器  | "html.parser" | bs4库                |
| lxml的html解析器 | "lxml"        | pip install lxml     |
| lxml的xml解析器  | "xml"         | pip install lxml     |
| html5lib的解析器 | "html5lib"    | pip install html5lib |



| 基本元素        | 说明                                              |
| --------------- | ------------------------------------------------- |
| Tag             | 标签  soup.\<tag\> # 多个相同标签，优先返回首个。 |
| Name            | 标签名 \<tag\>.name                               |
| Attributes      | 标签属性，字典形式 \<tag\>.attrs                  |
| NavigableString | 非属性字符串 \<tag\>.string                       |
| Comment         | 注释                                              |

实例：

```python
>>> from bs4 import BeautifulSoup
>>> soup = BeautifulSoup(demo, "html.parser") #demo是一个html文档(字符串) 通过demo = get("https://python123.io/ws/demo.html").text
>>> tag = soup.a
>>> tag.name
'a'
>>> tag.parent.name
'p'
>>> tag.attrs
{'href': 'http://www.icourse163.org/course/BIT-268001', 'class': ['py1'], 'id': 'link1'}
>>> tag.attrs['href']
'http://www.icourse163.org/course/BIT-268001'
>>> type(tag.attrs)
<class 'dict'>
>>> type(tag)
<class 'bs4.element.Tag'>
>>> tag.string
'Basic Python'
```

## 遍历HTML

> 才知道html竟然可以看成标签树形结构qwq

遍历方式： 上行遍历(fa -> son)，下行遍历(son -> fa)，平行遍历(**同父亲的**son之间，不是同深度的所有节点)

| 属性               | 说明                                     |
| ------------------ | ---------------------------------------- |
| .contents          | 子节点的列表，包含所有儿子节点           |
| .children          | 子节点的迭代类型，用于循环遍历儿子       |
| .descendants       | 子孙节点的迭代类型，包含所有子孙节点     |
| .parent            | 父亲节点标签                             |
| .parents           | 先辈标签的迭代类型，用于循环遍历先辈节点 |
| .next_sibling      | 返回html文本顺序的下一个平行节点标签     |
| .previous_sibling  | 上一个平行节点标签                       |
| .next_siblings     | 迭代类型，后续所有平行节点标签           |
| .previous_siblings | 迭代类型，前驱所有平行节点标签           |

<img src="https://s3.ax1x.com/2020/12/09/rPnRgK.jpg" width="500px">

下行遍历：

```python
>>> soup = BeautifulSoup(demo, "html.parser")
>>> soup.head
<head><title>This is a python demo page</title></head>
>>> soup.head.contents
[<title>This is a python demo page</title>]
>>> soup.body.contents[1]
<p class="title"><b>The demo python introduces several python courses.</b></p>
# 遍历儿子
for child in soup.body.children:
    print(child)
```

上行遍历：（接上段）

```python
>>> soup.title.parent
<head><title>This is a python demo page</title></head>
>>> soup.html.parent
# 打印了整个html文档
>>> soup.parent
# 空
# 遍历父亲
for parent in soup.a.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)
```

平行遍历：（接上段）

````python
>>> soup.a.next_sibling
' and '
>>> soup.a.next_sibling.next_sibling
<a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>
>>> soup.a.previous_sibling
'Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:\r\n'
>>> soup.a.previous_sibling.previous_sibling
# 空
# 遍历同上qwq
````

## HTML格式输出

> 如何让html页面更加"友好"地显示

### bs4库的pretify()方法

使用方法：` soup.prettify() # 添加空格和换行符用于打印 `

```python
>>> print(soup.prettify())
<html>
 <head>
  # 省略掉防止占篇幅qwq
 </head>
 <body>
  # 省略掉防止占篇幅qwq
 </body>
</html>
```

## 实例：提取HTML中所有链接

1. 找到所有\<a\>标签
2. 提取所有href后内容

```python
>>> import requests as rq
>>> from bs4 import BeautifulSoup
>>> url ="https://python123.io/ws/demo.html"
>>> r = rq.get(url)
>>> demo = r.text
>>> soup = BeautifulSoup(demo, "html.parser")
>>> for link in soup.find_all('a'):
	print(link.get('href'))
http://www.icourse163.org/course/BIT-268001
http://www.icourse163.org/course/BIT-1001870001
```

