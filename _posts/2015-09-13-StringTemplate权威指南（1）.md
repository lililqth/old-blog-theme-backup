## 前言
最近，我需要在自己的Compiler项目中生成目标代码，由于在语法分析阶段用到了Antlr，因此在代码生成阶段就很自然地选择了StringTemplate模板引擎。然而，除了官方文档，我竟然没有发现一份靠谱的StringTemplate v4版本的中文资料！因此，我决定将其所有英文文档翻译成中文，这是整个翻译计划的第一篇。  

## 简介
大多数需要生成代码或者其他文本的程序都是通过大量非结构化的print语句来实现的。造成这一结果的主要原因是缺少合适工具和方法。  
由于程序的输出并不是随机的文本，而是输出语言的句子，因此正确的方式是根据输出文法来生成文本。这和使用输入文法来描述输入文本是类似的。相比手写语法分析器（parser），大部分程序员都会使用parser生成工具比如说Antlr来自动生成parser。同样，我们也需要模板引擎（unparser）来自动生成代码生成工具。  
如果使用了Antlr作为parser生成器，那么同样出自Terence Parr之手的模板引擎StringTemplate无疑是最好的选择。值得一提的是，Antlr和StringTemplate的官网就是使用该模板引擎生成的。  
[Antlr官方网站](http://www.antlr.org/)  
[StringTemplate官方网站](http://www.stringtemplate.org/)  
模板引擎是通过模板生成文本的代码生成器，是专门用于生成文本的领域特定语言（domain-specific languages），本质上来讲就是『带有孔的文档』，文档中的孔称为属性（本文使用attribute代替）。  
Attribute可以代表一个字符串，一个整数，也可以代表一组属性，甚至是别的模板。
StringTemplate会将模板转换为一系列文本和属性的组合，其中的属性通常以< attribute-expression \>的形式出现（当然，也可以使用任何你想要的符号来包裹属性，比如说：$ attribute-expression \$）。StringTemplate会忽略除了属性外的所有文本，仅仅把它们当做普通文本来输出。为了渲染（请原谅我用了这么小资的词）模板生成文本，我们需要调用render方法。

{% highlight java%}
ST.render(); //java
{% endhighlight %}
StringTemplate支持Java，C#，Python，Objective-C等多种语言，但是对除了java的所有语言支持并不完整，有大量功能没有实现，所以在这里仅以java为例。

## 从Hello World开始
让我们从Hello World开始。  
下面是一个最简单的StringTemplate模板，由两个部分组成，一个简单文本（Hello）和一个属性（name）。  

{% highlight java%}
Hello, <name>
{% endhighlight %}

使用StringTemplate对其进行渲染。

{% highlight java%}
// Java文件
import org.stringtemplate.v4.*;
...
ST hello = new ST("Hello, <name>");
hello.add("name", "World");
System.out.println(hello.render());
{% endhighlight %}

> 模板引擎是MVC（Model-View-Controller）模式的一个典型例子，模板控制生成文本的样式（View），调
> 用模板引擎的代码会组织记录各种变量（Model），并将属性替换成文本（Controller）。

StringTemplate并不是一个十分庞大的引擎，系统或者说服务器，它是一个可以集成在其他程序中的一个小巧的类库，仅仅依赖于Antlr（用于解析模板）。

## 模板组
StringTemplate中有三个用于载入模板的方法：ST, STGroupDir, 和STGroupFile。通过这三个方法可以分别在代码中创建模板，从包含模板的目录中载入多个模板，从包含多个模板的文件中载入模板（Template Group 文件）。  
假设我们在tmp目录下有两个模板：

{% highlight java%}
///tmp/decl.st
decl(type, name, value) ::= "<type> <name><init(value)>;"
{% endhighlight %}
{% highlight java%}
///tmp/init.st
init(v) ::= "<if(v)> = <v><endif>"
{% endhighlight %}
我们可以通过STGroupDir对象来访问这些模板，之后用getInstanceOf()方法来得到某模板对象，并通过add()方法来对替换属性。

{% highlight java%}
//Java文件
STGroup group = new STGroupDir("/tmp");
ST st = group.getInstanceOf("decl");
st.add("type", "int");
st.add("name", "x");
st.add("value", 0);
String result = st.render(); // 输出 "int x = 0;"
{% endhighlight %}

这个例子展示了StringTemplate的核心方法，模板的定义非常类似于函数，只不过函数体变成了模板文本。decl模板生成了变量声明语句，有三个参数，其中两个参数可以直接被替换，最后一个init参数是对其他模板的引用，**如果该变量的声明并没有初始化，就不用传入 value 参数，这样就会生成 int x; 宏代码的存在，可以使模板被动态地组合。** 另外，模板可以直接访问子模板中的任何一个属性。
通常情况下，将模板放在同一个文件中会比较方便，这个模板集合文件称为（Template-Group File），在代码中仅仅需要将 STGroupDir 换为 STGroupFile 即可。

{% highlight java%}
///tmp/test.stg
decl(type, name, value) ::= "<type> <name><init(value)>;"
init(v) ::= "<if(v)> = <v><endif>"
{% endhighlight %}
{% highlight java%}
// Java文件
STGroup group = new STGroupFile("/tmp/test.stg");
ST st = group.getInstanceOf("decl");
st.add("type", "int");
st.add("name", "x");
st.add("value", 0);
String result = st.render(); // yields "int x = 0;"
{% endhighlight %}

## 访问对象的成员
在模板的属性字段中，我们可以直接访问其他对象的成员。下面是一个user类。  

{% highlight java%}
public static class User {
    public int id; // template can directly access via u.id
    private String name; // template can't access this
    public User(int id, String name) { this.id = id; this.name = name; }
    public boolean isManager() { return true; } // u.manager
    public boolean hasParkingSpot() { return true; } // u.parkingSpot
    public String getName() { return name; } // u.name
    public String toString() { return id+":"+name; } // u
}
{% endhighlight %}

我们可以像其他对象一样（string等预先定义好的），将user对象赋给模板属性。而且在模板属性中可以通过'.'来访问对象的成员。当写下"o.p"的时候，StringTemplate会在对象o中寻找成员p。然而寻找的方式稍微有点特殊。StringTemplate会在类的定义中优先寻找"getP()", "isP()", "hasP()"方法。如果找不到类似的方法，才会寻找名为p的成员。在下面的例子中，我们寻找对象u的id和name成员。  
注意：这个例子中ST的第二和第三个参数分别代表和属性字段的前分界符和后分界符。在这里规定前分界符和后分界符均为'\$'这在生成HTML的时候尤其有用。  

{% highlight java%}
//Java代码
ST st = new ST("<b>$u.id$</b>: $u.name$", '$', '$');
st.add("u", new User(999, "parrt"));
String result = st.render(); 
//输出："<b>999</b>: parrt"
{% endhighlight %}

这个例子中，u.id是直接通过访问成员id得到的，u.name是通过u.getName()得到的。

## 将属性替换成其他模板
接下来我们将对模板属性的渲染方式作更加细致的研究。属性并不区分单值和多值属性。例如：如果我们使用 add() 方法将 "parrt" 赋给 "<name\>" 属性，name 会被渲染成 parrt ；如果我们两次调用 add() 方法将"parrt"和"tombu"赋给name属性，name会被渲染为"parrttombu"。  
为了能够区分每一次add的元素，可以使用 "separator" 选项： <name; separator=\",\"\> 渲染的结果是： "parrt,tombu"  
如果要调整每一个元素的表现形式，比如说要将名字用方括号包裹，我们可以通过子模板来定义属性的表现形式：  

{% highlight java%}
// apply bracket template to each name
test(name) ::= "<name:bracket()>"
// surround parameter with square brackets
bracket(x) ::= "[<x>]"
{% endhighlight %}

经过两次add()赋值后，上面的模板将会被渲染为 "[parrt],[tombu]"  
StringTemplate并不关心我们传给属性的对象的类型，例如，我们可以将一系列User对象传入模板，模板引擎在渲染时会直接调用对象的toString()方法。下面是一个例子：  

{% highlight java%}
// 模板
// apply bracket template to each name
test(name) ::= "<name:bracket();separator=\",\">"
// surround parameter with square brackets
bracket(x) ::= "[<x>]"
{% endhighlight %}
{% highlight java%}
// Java文件
st.add("name", new User(1000, "parrt"));
st.add("name", new User(999, "tumbo"));
// 渲染结果是[1000:parrt],[999,tumbo]
{% endhighlight %}

如果该对象没有重写toString()方法，由于java中所有类继承自Object类，模板引擎就会调用父类的toString()方法。该方法会返回该类的"类名+@+hashcode"值。  
有的时候为一个很小的或者只出现了一次的模板定义一个单独的模板会显得比较繁琐，为了简化操作，我们可以使用匿名模板（或者说子模板）。子模板没有自己的名称，其他方面和普通的模板完全一致。我们可以将上面的例子重写成如下的形式：  

{% highlight java%}
test(name) ::= "<name:{x | [<x>]}; separator=\", \">"

{% endhighlight %}


匿名模板{x|<x\>}是bracket()模板的匿名形式。参数名被单独地用\"\|\"分隔开来。

