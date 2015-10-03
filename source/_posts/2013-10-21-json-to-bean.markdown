---
layout: post
title: "JSON和bean互转"
date: 2013-10-21 09:32
comment: true
categories: java
---
需要用到的类：

	net.sf.json.JSONObject  
在包`json-lib-2.3-jdk15`中

<!-- more -->

maven依赖：
``` 
<dependency>
	<groupId>net.sf.json-lib</groupId>
	<artifactId>json-lib</artifactId>
	<version>2.4</version>
	<classifier>jdk15</classifier>
</dependency>
```
将bean转换成JSONObject，调用`JSONObject.fromObject(object)`
如：

	JSONObject jsonObject = JSONObject.fromObject(user);

将JSONObject转换成bean，调用`JSONObject.toBean(jsonObject, beanClass)`
如：

	User user = (User) JSONObject.toBean(jsonObject, User.class);
