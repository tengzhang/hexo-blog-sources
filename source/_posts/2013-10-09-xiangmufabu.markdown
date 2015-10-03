---
layout: post
title: "项目发布"
date: 2013-10-09 09:40
comment: true
categories: 
---
1. 去发布目录下建立相应的文件夹  
2. 更改发布脚本  
3. 执行脚本  
4. 接下来就分两种情况了  

<!-- more -->

4.1.静态网站
修改`nginx`配置文件`nginx.conf`  
在最后加上如下代码：  
```
server {
		listen       192.168.1.xx:80;
		server_name  xxx.net;
		charset utf-8;

		location / {
			root   /leave/xx/deploy/xx;

			autoindex on;
			autoindex_exact_size off;
			autoindex_localtime on;

			index  index.html;
		}
   }
#ps:xx根据实际情况修改
```
4.2. java web网站
(1)修改`nginx`配置文件`nginx.conf`  
在最后加上如下代码：
```  
server {
		listen       192.168.1.x:80;
		server_name  xxx.net;
		charset utf-8;

		location / {
				proxy_set_header   Host             $host;
				proxy_set_header   X-Real-IP        $remote_addr;
				proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
				client_max_body_size       1024m;
				client_body_buffer_size    128k;
				client_body_temp_path      data/client_body_temp;
				proxy_connect_timeout      90;
				proxy_send_timeout         90;
				proxy_read_timeout         90;
				proxy_buffer_size          4k;
				proxy_buffers              4 32k;
				proxy_busy_buffers_size    64k;
				proxy_temp_file_write_size 64k;
				proxy_temp_path            data/proxy_temp;
				proxy_pass http://127.0.0.1:8080;
		}
   }
#ps:xx根据实际情况修改
```
(2)修改tomcat配置文件server.xml
加上如下代码：
```
<Host name="xxx.net"  appBase="webapps" unpackWARs="true" autoDeploy="true">
	<Context path="" docBase="/xxx/deploy/xxx" debug="0" reloadable="true" crossContext="true" />
</Host>
#ps:xx根据实际情况修改
```
5. 平滑重启nginx
```
ps -aux | grep nginx
// 找到nginx master的pid
kill -HUP pid
```
6. 如果是java web，还需重启tomcat
7. 更改路由
```
system-view
ip host xxx.net 192.168.1.xx
```
