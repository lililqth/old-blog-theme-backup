---
layout: post
title: "使用ElementTree解析XML"
date: 2015-03-18 20:00:00
---

elementTree是Python的一个XML解析包，可以十分方便的在XML文件中查找，删除，移动结点或者子树。

## 引用包

{% highlight python%}
try:
    import xml.etree.cElementTree as ET
except ImportError:
    import xml.etree.ElementTree as ET
{% endhighlight %}

## 基本操作
**从文件读取：**

经过parse操作后，就可以得到一棵语法树

{% highlight python%}
tree = ET.parse(fileName)
{% endhighlight %}

**获取根节点**

{% highlight python%}
root = tree.getroot()
{% endhighlight %}

**删除子节点**


{% highlight python%}
root = tree.getroot()
item.remove(subItem)
{% endhighlight %}

**添加子节点**


{% highlight python%}
root = tree.getroot()
# 添加一个子节点
item.append(subItem)

# 添加多个子节点
item.extend(subItemList)
{% endhighlight %}




**查找节点**


{% highlight python%}
root = tree.getroot()
# 查找符合匹配的第一个结点
item.find("Path")
# 查找符合要求的所有结点
item.findall("Path")
{% endhighlight %}
ElementTree一定程度上支持xpath，但是所支持的这些特性我认为已经足够用，下面是常用的几种用法：


{% highlight python%}
root = tree.getroot()
# 递归的查找所有符合要求的结点
item.find(".//Name")
# 其中【2】表示符合要求的第二个Chart
item.find("Model/Chart[2]/state")
# 查找Name属性为OUTPUT的Model
item.find("Model[@Name='OUTPUT']")
{% endhighlight %}

下图的表格中是完整的用法:

|语法|作用|
|---------------|--------------|
|*|查找所有的孩子结点，比如："*/egg":查找所有叫做egg的孙子结点|
|.|表示当前路径，一般用在路径的开头表明这是一个相对路径。不加这个点直接写其实也是可以的|
|//|递归地查找所有的孩子结点|
|..|表示父结点|
|[@attrib]|查找带有attrib属性的所有结点|
|[@attrib='value']|查找attrib=value的结点|
|[tag]|查找所有的包含叫做tag的孩子的结点，只能有一个tag，只能向下查找一层|
|[position]|查找在某个位置的结点Chart[2]查找第二个chart结点，chart.last()表示最后一个结点|

**移动结点**

要移动结点可以将find到的结点直接append到目标位置的父节点上，可以将该结点连同所有子节点都递归地移动到目标结点:


{% highlight python%}
root = tree.getroot()
element = item.find("Name")
dst.append("elememt")
{% endhighlight %}

## 获取结点信息
每个element对象都具有以下属性：
1. tag：string对象，表示数据代表的种类。
2. attrib：dictionary对象，表示附有的属性。
3. text：string对象，表示element的内容。
4. tail：string对象，表示element闭合之后的尾迹。
5. 若干子元素（child elements）。

{% highlight xml%}
<tag attrib1=1>text</tag>tail
  1     2        3         4
{% endhighlight %}

如果要获取某个对象的这些属性也十分简单:

{% highlight python%}
# 获取tag
item.tag
# 获取attrib
item.attrib
...
{% endhighlight %}

## 保存
以上进行的所有处理都是在内部数据结构上进行的，要使之生效必须保存到磁盘上：  

{% highlight python%}
tree.write(fileName)
{% endhighlight %}
