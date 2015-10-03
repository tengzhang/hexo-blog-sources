---
layout: post
title: "structs2返回json"
date: 2013-10-30 09:28
comment: true
categories: 
---
structs2返回本身支持返回json，只需要配置即可，我们也可以自己把json串写入servlet输出流中。

<!-- more -->

1.structs2本身对json的支持
在action中对需要返回的值进行设置，实体、java自带类型都可以。
如：

``` java
private User user;
private result result;

/*   setter and getter   */
public String json() {
    user.setUserName("iris");
    result = "success";
    return SUCCESS;
}
```
在配置文件中配置result的type为json

```
<result-types>
    <result-type name="json" class="org.apache.struts2.json.JSONResult"/>
</result-types>

<action name="hello_*" class="action.FirstAction" method="{1}">
    <result name="success" type="json"></result>
</action>
```
注意result-types那段一定要加上，不加那个的就必须让action所在的package继承`json-default`  
2.把json数据写入servlet输出流中  
在action中讲json数据直接写入servlet输出流中，action中的方法返回void  
如：

``` java
public void newjsontest() {
    HttpServletResponse response = ServletActionContext.getResponse();
    PrintWriter out = null;
    try {
        out = response.getWriter();
	User user = new User();
	user.setUsername("iris");
	user.setPassword("iris");
	JSONObject jsonObject = JSONObject.fromObject(user);
	out.print(jsonObject.toString());
	out.flush();
    } catch (IOException e) {
	e.printStackTrace();
    } finally {
	if(out != null) {
	    try {
    	        out.close();
    	    } catch (Exception e) {
	        e.printStackTrace();
	    }
        }
    }
}
```
这里用到了`net.sf.json.JSONObject`这个包，将实体直接转换成json，也可以不用这个包，直接手动拼json串，然后`out.write(jsonStr)`。
