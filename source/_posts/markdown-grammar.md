title: 'markdown语法简介'
date: 2014-07-07 20:41:20
tags:
---
本文摘抄自[http://wowubuntu.com/markdown/](http://wowubuntu.com/markdown/)，旨在联系一下markdown的语法，同时也向大家介绍一下markdown的语法，方便大家和自己查阅。

___

Markdown 语法说明（简体中文版）
================
*	[概述](#overview)
	*	[宗旨](#philosophy)
	*	[兼容 HTML](#html)
	*	[特殊字符自动转换](#autoescape)
*	[区块元素](#block)
	*	[段落和换行](#p)
	*	[标题](#header)
	*	[区块引用](#blockquote)
	*	[列表](#list)
	*	[代码区块](#precode)
	*	[分割线](#hr)
*	[区段元素](#span)
	*	[链接](#link)
	*	[强调](#em)
	*	[代码](#code)
	*	[图片](#img)
*	[其它](#misc)
	*	[反斜杠](#backslash)
	*	[自动连接](#autolink)
*		[Markdown 免费编辑器](#editor)

<!-- more -->

* * *

<h2 id="overview">概述</h2>

<h3 id="philosophy">宗旨</h3>

Markdown的目标是实现「易读易写」。

可读性，无论如何，都是最重要的。一份使用Markdown格式撰写的文件应该可以直接以纯文本发布，并且看起来不会像是由许多标签或是格式指令所构成。Markdown语法受到一些既有text-to-HTML格式的影响，包括[Setext] [1]、[atx] [2]、[Textile] [3]、[reStructuredText] [4]、[Grutatext] [5]和[EtText] [6]，而最大灵感来源其实是纯文本电子邮件的格式。

  [1]: http://docutils.sourceforge.net/mirror/setext.html
  [2]: http://www.aaronsw.com/2002/atx/
  [3]: http://textism.com/tools/textile/
  [4]: http://docutils.sourceforge.net/rst.html
  [5]: http://www.triptico.com/software/grutatxt.html
  [6]: http://ettext.taint.org/doc/

总之，Markdown的语法全由一些符号所组成，这些符号经过精挑细选，其作用一目了然。比如：在文字两旁加上星号，看起来就像\*强调\*。Markdown的列表看起来，嗯，就是列表。Markdown的区块引用看起来就真的像是引用一段文字，就像你曾在电子邮件中见过的那样。

<h3 id="html">兼容 HTML</h3>

Markdown语法的目标是：成为一种适用于网络的*书写*语言。

Markdown不是想要取代HTML，甚至也没有要和它相近，他的语法种类很少，只对应HTML标记的一小部分。Markdown的构想*不是*要使得HTML文档更容易书写。在我看来，HTML已经很容易书写了。Markdown的理念是，能让文档更容易读、写和随意改。HTML是一种*发布*的格式，Markdown是一种*书写*的格式。就这样，Markdown的格式语法只涵盖纯文本可以涵盖的范围。

不在Markdown涵盖范围之内的标签，都可以直接在文档里面用HTMl撰写。不需要额外标注这是HTML或是Markdown；只要直接加标签就可以了。

要制约的只有一些HTML区块元素--比如`<div>`、`<table>`、`<pre>`、`<p>`等标签，必须在前后加上空行与其它内容区隔开，还要求它们的开始标签与结尾标签不能用制表符或空格来缩进。Markdown的生成器有足够只能，不会在HTML区块标签外加上不必要的`<p>`标签。

例子如下，在 Markdown 文件里加上一段 HTML 表格：

    这是一个普通段落。

    <table>
        <tr>
            <td>Foo</td>
        </tr>
    </table>

    这是另一个普通段落。

请注意，在 HTML 区块标签间的 Markdown 格式语法将不会被处理。比如，你在 HTML 区块内使用 Markdown 样式的`*强调*`会没有效果。

HTML的区段（行内）标签如`<span>`、`<cite>`、`<del>`可以在Markdown的段落、列表或是标题里随意使用。依照个人习惯，甚至可以不用Markdown格式，而直接此阿勇HTML标签来格式化。举例说明：如果比较喜欢HTML的`<a>`或`<img>`标签，可以直接使用这些标签，而不用Markdown提供的链接或是图像标签语法。

和处在HTML区块标签件不同，Markdown语法在HTML区段标签间是有效的。