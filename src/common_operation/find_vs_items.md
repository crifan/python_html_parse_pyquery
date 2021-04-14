# find() vs items()

## 概述

* `PyQuery.find()`：返回的是`lxml`的`element`的`generator`
* `PyQuery.items()`：返回的是`PyQuery`的`generator`

## 详解

* find
  * 官网文档：[pyquery.pyquery.PyQuery.find](https://pythonhosted.org/pyquery/api.html#pyquery.pyquery.PyQuery.find)
    > PyQuery.find(selector)
    > 
    >    Find elements using selector traversing down from self:
    ```python
    >>> m = '<p><span><em>Whoah!</em></span></p><p><em> there</em></p>'
    >>> d = PyQuery(m)
    >>> d('p').find('em')
    [<em>, <em>]
    >>> d('p').eq(1).find('em')
    [<em>]
    ```
  *  -> `find()`返回的是`element`元素=`lxml.html.HtmlElement`的`generator`
* items
  * 官网文档：[pyquery.pyquery.PyQuery.items](https://pythonhosted.org/pyquery/api.html#pyquery.pyquery.PyQuery.items)
    > PyQuery.items(selector=None)
    > 
    >    Iter over elements. Return PyQuery objects:
    ```python
    >>> d = PyQuery('<div><span>foo</span><span>bar</span></div>')
    >>> [i.text() for i in d.items('span')]
    ['foo', 'bar']
    >>> [i.text() for i in d('span').items()]
    ['foo', 'bar']
    ```
  * -> `items()`返回的是`PyQuery`=`pyquery.pyquery.PyQuery`的`generator`

### 举例

```xml
    <ul class="rank-list-ul" 0>

      <li id="s3170">
...
      </li>

      <li id="s692">
...
      </li>
```

#### find()

如果是`find()`：

```python
carSeriesDocGenerator = merchantRankDoc.find("li[id*='s']")
print("type(carSeriesDocGenerator)=%s" % type(carSeriesDocGenerator))
```

则返回的是`PyQuery`

```bash
type(carSeriesDocGenerator)=<class 'pyquery.pyquery.PyQuery'>
```

然后`generator`转换成`list`后：

```python
carSeriesDocList = list(carSeriesDocGenerator)
for curSeriesIdx, eachCarSeriesDoc in enumerate(carSeriesDocList):
    print("type(eachCarSeriesDoc)=%s" % type(eachCarSeriesDoc))
```

每个元素是：`lxml.html.HtmlElement`

```bash
type(eachCarSeriesDoc)=<class 'lxml.html.HtmlElement'>
```

#### items()

如果换成`items()`

```python
carSeriesDocGenerator = merchantRankDoc.items("li[id*='s']")
print("type(carSeriesDocGenerator)=%s" % type(carSeriesDocGenerator))
```

则返回的是`generator`：

```bash
type(carSeriesDocGenerator)=<class 'generator'>
```

然后`generator`转换成`list`后：

```python
carSeriesDocList = list(carSeriesDocGenerator)
for curSeriesIdx, eachCarSeriesDoc in enumerate(carSeriesDocList):
    print("type(eachCarSeriesDoc)=%s" % type(eachCarSeriesDoc))
```

每个元素是：`pyquery.pyquery.PyQuery`

```bash
type(eachCarSeriesDoc)=<class 'pyquery.pyquery.PyQuery'>
```
