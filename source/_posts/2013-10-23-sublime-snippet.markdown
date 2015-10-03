---
layout: post
title: "sublime利用snippet生成代码模版"
date: 2013-10-23 14:20
comment: true
categories: sublime
---
`tools->New Snippet`新建一个snippet，保存为**.sublime-snippet。  
内容如下：  

<!-- more -->

``` html
<snippet>
    <content>
    <![CDATA[
<!DOCTYPE HTML> 
<html> 
<head> 
    <meta charset="utf-8"> 
    <title>${1}</title> 
</head>
<body>
    ${1}
</body>
</html>
    ]]>
    </content>
    <!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
    <tabTrigger>myhtml</tabTrigger>
    <!-- Optional: Set a scope to limit where the snippet will trigger -->
    <!-- <scope>source.python</scope> -->
</snippet>sublime-snippet
```
重启sublime，新建文件输入myhtml，然后按`tab`。
