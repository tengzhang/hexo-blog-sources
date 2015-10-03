title: CentOS下安装Redmine
date: 2014-09-09 14:37:15
tags: [linux, redmine]
comment: true
toc: true
share: true
---
Redmine的安装主要分为两部分：
1. Ruby+Rails的安装
2. Redmine的安装
先说一下系统环境，我安装的系统是CentOS 5，选择的Redmine版本是2.5.2，Ruby的版本是1.9.3，Rails的版本是3.2.19。
下面从以上两部分进行说明。

### Ruby+Rails的安装
Ruby的安装有3中方式：
1. 直接`yum install ruby`
2. 源码编译安装
3. 采用RVM安装
以上3种方式我都尝试了，最后采用了RVM安装这种方式，这种方式也是最简单的。

<!-- more -->

1. 下载RVM并且安装
``` sh
curl -L https://get.rvm.io | bash -s stable
```

2. 安装一些依赖库，编译安装其它软件或者库的时候会用到
``` sh
yum install zlib zlib-devel sqlite-devel
yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel
```

3. 安装openssl，安装ruby的时候会使用到
``` sh
rvm pkg install openssl 
```

4. 指定openssl的位置来安装ruby
``` sh
rvm install 1.9.3 --with-openssl-dir=/usr/local/rvm/usr
```

5. 指定使用Ruby版本
``` sh
rvm use 1.9.2
```

6. 安装Rails
``` sh
gem install rails
```

经过以上步骤，Ruby和Rails就安装完毕了。
安装完Ruby之后，建议更改一下gem的镜像地址，原因你懂的。可以改到淘宝的gem镜像，参考一下网址：[http://ruby.taobao.org/](http://ruby.taobao.org)。上面有更改gem镜像的详细方法。


### Redmine的安装
最推荐的做法是参考官方wiki（http://www.redmine.org/projects/redmine/wiki/RedmineInstall）
1. 准备Redmine需要用到的数据库环境
	1.1. 修改Redmine数据库配置文件
	进入Redmine目录
``` sh 
cp config/database.yml.example config/database.yml  
vi database.yml  
```
按照下面的内容进行修改
```
production:  
adapter: mysql2  
database: db_redmine  
host: localhost  
username: root  
password: <你的mysql密码>  
ps: 其中的一些值需要根据实际情况
```

	1.2. 进入Mysql创建数据库
``` sql
create database redmine character set utf8;
```

2. 初始化Redmine数据库环境
``` sh
rake db:migrate RAILS_ENV=production
```
执行此命令的时候会出现以下错误：
```
Could not find gem 'mocha (~> 1.0.0) ruby' in the gems available on this machine.
Run `bundle install` to install missing gems.
```
报这个错误的原因是因为缺少`mocha`这个gem，类似的还有很多gem都缺失。解决方法就是根据提示一个一个安装（类似于`gem install mocha -v=1.0.0`这样的命令），另一种解决办法是使用`bundle install`自动安装所缺少的gem，推荐使用这种方式，这种方式会自动帮你下载所依赖的gem，并且还能解决它们之间的版本冲突问题。
通过`bundle install`命令基本可以安装上所有依赖的gem，除了一个，`rmagick`，这个gem还需要依赖`ImageMagick`，他是用于处理一些图片转换之类的工作的。由于`rmagick`需要依赖`ImageMagick`，所以使用`bundle install`会安装失败，需要现在系统中安装`ImageMagick`。
解决方法如下：
	2.1. 如果你的CentOS版本比较新，可直接采用yum安装
	``` sh
	yum install ImageMagick-devel
	```

	2.2. 源码编译安装
	去官网下载源码(http://www.imagemagick.org/script/install-source.php#unix)
	上面有安装教程，建议安装上面的步骤进行安装。
	下面列出我安装时执行的命令：
	``` sh
	tar xvzf ImageMagick.tar.gz
	cd ImageMagick-6.8.9
	./configure
	make
	make install
	```

	经过上面的步骤将gem都安装完之后，还需要再次执行`rake db:migrate RAILS_ENV=production`，这一步的主要作用是在数据库中生成Redmine需要的数据库表。

3. 生成Redmine的Session存储
``` sh
rake generate_secret_token
```

4. 启动Redmine
``` sh
ruby script/rails server webrick -e production
```
这条命令不是后台运行，建议使用如下命令，让redmie在后台运行
``` sh
ruby script/rails server -e production -d > /dev/null 2>&1 &
```
可将上述命令加入/etc/local中，使得Redmine可以在系统启动时就启动。

在使用过程中可能会遇到如下错误：
``` sh
ActionView::Template::Error (incompatible character encodings: UTF-8 and ASCII-8BIT)
```
解决方法如下：
``` sh
rake assets:precompile
然后重启rails server。
```

### 插件安装
Redmine还可以安装一些插件，来帮助我们进行项目管理。
我们现在安装的插件有crumpm、code review、scrum、Aginle。
code review下载地址:[http://www.redmine.org/plugins/codereview](http://www.redmine.org/plugins/codereview)
scrum下载地址:[https://redmine.ociotec.com/projects/redmine-plugin-scrum](https://redmine.ociotec.com/projects/redmine-plugin-scrum)
Aginle下载地址:[http://www.redmine.org/plugins/redmine_agile](http://www.redmine.org/plugins/redmine_agile)

插件安装步骤:
1. 下载插件，解压到Redmine安装目录下的plugins文件夹下
2. 在Redmine安装目录下执行:
``` sh
rake redmine:plugins:migrate RAILS_ENV=production
```
3. 重启Redmine

安装过程的参考资料如下：
1. http://lxiaodao.iteye.com/blog/1579992
2. http://note.youdao.com/share/?id=610d4aea90c5d7281b61dfcb1cd41906&type=note
3. http://blog.sina.com.cn/s/blog_8254427901016z1l.html
