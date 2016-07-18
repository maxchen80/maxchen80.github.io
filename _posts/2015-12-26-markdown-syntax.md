---
layout: post
title: Markdown语法教程
---

Markdown作为一个排版语言和工具，应用得是越来越广泛。在互联网上传统编辑文章的方式大部分是借助富文本编辑器，而现在越来越多的开始支持Markdown，尤其是github使用Markdown生产的文档页面如此简洁大方，吸引了大量程序员开始使用Markdown。而目前印象笔记和有道云笔记也均开始支持Markdown，所以是时候让我们了解一下Markdown到底是啥，有什么神奇的魔力吸引这么多的大型平台和大量用户的支持。以下教程翻译自Markdown官网，如有不准确的地方欢迎大家讨论。

Markdown，将文本转换为html的工具。Markdown允许你使用一个容易编写、容易阅读的纯文本格式，然后将其转换为XHTML（HTML）。
因此，Markdown干了2件事：
* 一个纯文本格式的语法
* 一个Perl编写的软件，将该纯文本格式转换为HTML。

Markdown的首要设计目标就是易读。Markdown格式的文档应该是像纯文本一样，而不是看起来像被打上了标记或格式化命令。Markdown语法受到了几个已有的文本转HTML格式的影响，而最大的一个灵感来源是纯文本电子邮件格式。

### Markdown 语法
#### 段落和换行
三大赛复赛地方vsdvsdvsdvzxvfdfsd

#### 标题
标题使用1~6个#开头，对应的标题级别是1~6。例如：
```
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```

#### 引用
引用使用电子邮件风格的`>`字符。例如：
```
> 这里是一段引用文字
```
> 这里是一段引用文字

#### 列表
Markdown支持有序列表和无序列表。
无序列表使用*/+/-作为列表标记：
```
* 红色
* 绿色
* 蓝色
```
* 红色
* 绿色
* 蓝色

有序列表使用数字紧跟一个英文句号：
```
1. 红色
2. 绿色
3. 蓝色
```
1. 红色
2. 绿色
3. 蓝色

#### 代码块
在Markdown中生成一个代码块，只需要简单的在每一行缩进至少4个空格或者1个tab：
```
这是一段代码块。
```
这是一段代码块。

#### 水平线
在同一行里使用至少3个*/-生成一条分隔符,或者使用这一行下面的下划线：
```
***
```
***

#### 链接
Markdown支持两种样式的链接：内联和参考。
在这两种风格里，链接文本都是由[方括号]分隔。
创建一个内联的链接，在链接文本的结束方括号之后立即使用一组正则括号。在括号内，放入网站链接，以及一个可选的标题，使用双引号。例如：
```
This is [腾讯网](http://www.qq.com/ "腾讯网") inline link.
```
This is [腾讯网](http://www.qq.com/ "腾讯网") inline link.
参考样式的链接，在链接文本后使用第2组方括号，里面是你确认链接的标签：
```
This is [腾讯网][qq] reference-style link.
```
This is [腾讯网][qq] reference-style link.
然后，在文档里的任意地方，定义你的链接标签：
```
[qq]: http://www.qq.com/  "腾讯网"
```

#### 重点
Markdown以`*`和`_`作为重点标识。被1个`*`或`_`包围的文本会被包装成HTML的`<em>`标签，被2个`*`或`_`包围的文本会被包装成HTML的`<strong>`标签：
```
*single asterisks*
_single underscores_
```
*single asterisks*
_single underscores_
```
**double asterisks**
__double underscores__
```
**double asterisks**
__double underscores__

#### 代码
表示一小段行内代码，用反引号(\`)引用。与预先格式化的代码块不同，代码段显示正常段内的代码。例如：
```
Use the `printf()` function.
```
Use the `printf()` function.

#### 图片
Markdown使用的是与链接语法类似的图片语法，允许两种样式：内联和参考。
内联图片语法看起来像这样：
```
![Alt text](https://sp0.baidu.com/-aYHfD0a2gU2pMbgoY3K/resource/1165d35e6ba1d5bb5620901461808709.jpg "Optional title")
```
![Alt text](https://sp0.baidu.com/-aYHfD0a2gU2pMbgoY3K/resource/1165d35e6ba1d5bb5620901461808709.jpg "Optional title")
参考图片语法看起来像这样：
```
![Alt text][id]
```
id是一个定义的图像引用的名称，使用与链接引用相同的语法定义：
```
[id]: https://sp0.baidu.com/-aYHfD0a2gU2pMbgoY3K/resource/1165d35e6ba1d5bb5620901461808709.jpg "Optional title attribute"
```
截至发稿，Markdown没有语法指定图像的尺寸；如果这对你很重要，你可以简单地使用常规的HTML`<IMG>`标签。

#### 自动链接
Markdown支持一种简单风格的“自动链接”来处理URL和邮箱地址：只需要使用尖括号将URL或邮箱包围起来。这意味这你像显示的文本就是实际链接内容，且它本身也是一个可用的链接，你可以这样做：
```
<http://www.qq.com/>
```
<http://www.qq.com/>

#### 反斜杠
Markdown允许使用反斜杠来生产在Markdown语法中有特殊含义的字符。例如，你想使用星号围绕文字（而不是HTML的`<strong>`标签），你可以在星号的前面使用反斜杠：
```
\*literal asterisks\*
```
\*literal asterisks\*
Markdown支持在下列字符前面使用反斜杠：
```
\   backslash
`   backtick
*   asterisk
_   underscore
{}  curly braces
[]  square brackets
()  parentheses
#   hash mark
+   plus sign
-   minus sign (hyphen)
.   dot
!   exclamation mark
```

###markdown官网
https://daringfireball.net/projects/markdown/
