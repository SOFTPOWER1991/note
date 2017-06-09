本文主要讲解BeautifulSoup的使用，主要包括如下几点：

1. BeautifulSoup是什么？
2. 用来解决什么问题？
3. 怎么用？
4. 对象的种类
5. BeautifulSoup提供的功能


下面来逐个讲解。

# 1. BeautifulSoup 是什么？

Beautiful Soup 是一个可以从HTML或XML文件中提取数据的Python库.

# 2. 它用来解决什么问题？

它能够通过你喜欢的转换器实现惯用的文档导航,查找,修改文档的方式.在爬虫将数据趴下来后，使用该框架进行数据解析。

# 3. 怎么用？

## 3.1 安装BeautifulSoup

> pip3 install beautifulsoup4

## 3.2 如何使用

```
from bs4 import BeautifulSoup

soup = BeautifulSoup(open("index.html"))

soup = BeautifulSoup("<html>data</html>")
```
Beautiful Soup选择最合适的解析器来解析这段文档,如果手动指定解析器那么Beautiful Soup会选择指定的解析器来解析文档.要在 BeautifulSoup 构造方法中加入第二个参数 “xml”:

```
soup = BeautifulSoup(markup, "xml")
```

# 4. 对象的种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` .

# 5. BeautifulSoup提供的功能

使用文档：[Beautiful Soup 4.2.0 文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)
## 5.1 格式化后浏览数据
## 5.2 访问Tag
## 5.3 访问属性
## 5.4 获取文本
## 5.5 注释处理
## 5.6 搜索
## 5.7 css选择器






