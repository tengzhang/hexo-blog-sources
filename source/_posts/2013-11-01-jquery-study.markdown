---
layout: post
title: "jquery使用笔记"
date: 2013-11-01 10:43
comment: true
categories: jquery
---
1. ajax请求，注意最后一个值的地方没有逗号
``` js
$.ajax({  
	type: "POST",
	url: "mystructs/hello_jsonTest.action",
	data: {
		number: $("#number").val(),
		userId: $('#userId').val() 
	},
	dataType: "text",  //ajax返回值设置为text（json格式也可用它返回，可打印出结果，也可设置成json）
	success: function(json) {
		var res = $.parseJSON(json);
		alert(res.result);
	},
	error: function(json) {
		alert(json);
	}
});
```

<!-- more -->

2. jquery改变值
``` js
$('#id').val('value');
$('#id').html('value');
```
3. jquery获取radio中被选中的那个元素
``` js
var val = $('input:radio[name="ra"]:checked').val();
if(val != null) {
	alert(val);
} else {        
	...
}
```
判读radio是否被选中
``` js
$('input:radio[name="ra"]').each(function() {
	//alert($(this).val());
	if($(this).attr("checked") == "checked") {
		alert($(this).val());
	}
});
```
4. 获取选中的checkbox
``` js
$('input[name="chk"]:checked').each(function(){
	alert($(this).val());
});
```
判读checkbox是否被选中
``` js
$('input:checkbox[name="chk"]').each(function() {
	if($(this).attr("checked") == "checked") {
		alert($(this).val());
	}
});
```
