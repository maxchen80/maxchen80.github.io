---
layout: post
title: GitHub Flavored Markdown语法
---

上一篇文章介绍的标准的Markdown语法。学会了这些语法我们基本就能编写日常文档了。但是如果你是程序员或者是编写学术型文档，那么标准的Markdown语法可能就无法满足需求了。比如表格、数学表达式、流程图等等。这个时候我们需要一些对Markdown的扩展支持。当然没有任何一个扩展可以支持所有的需求，针对程序员而言，github对Markdown的扩展支持基本能够满足日常需求。

github.com使用自己独有的Markdown语法提供了一些有用的功能，使得更容易在github.com上进行工作。
对于不使用github的用户而言，学习GFM也是有用的，因为事实上目前主流的Markdown编辑器，都支持了GFM语法。
####语法高亮
下面是一个使用GFM进行语法高亮的例子：
	```javascript
	function fancyAlert(arg) {
      if(arg) {
        $.facebox({div:'#foo'})
      }
    }
	```
```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```
####任务列表
```
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item
```
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item
####表格
笑嘻嘻笑嘻嘻笑嘻嘻笑嘻嘻
Title01 | Title02
------- | -------
1234567 | abcdefg
1234567 | abcdefg
####SHA references
####Issue references within a repository
