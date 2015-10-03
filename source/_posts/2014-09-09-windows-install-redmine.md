title: Windows下安装Redmine
date: 2014-09-09 14:39:16
tags:
comment: true
toc: true
share: true
---
### 准备工作
Redmine官网:[http://www.redmine.org/](http://www.redmine.org/)
Ruby官网:[http://rubyforge.org/](http://rubyforge.org/)

1. 下载railsinstaller-2.2.3.exe
    这是Ruby环境的一键安装包，安装上它就可以安装上全部的Ruby环境。
    官网:[http://railsinstaller.org/en](http://railsinstaller.org/en)
    此安装包包含如下内容：
    Ruby 1.9.3
Rails 3.2
Bundler
Git
Sqlite
TinyTDS
SQL Server Support
DevKit

<!-- more -->

2. 下载ImageMagick，这个是图型生成工具，redmine用于生成pdf等内容，也可以不安装。安装说明在这里
我下载的是ImageMagick-6.8.4-0-Q16-x86-dll.exe
下载地址:[http://download.csdn.net/detail/lqt0307/5184966](http://download.csdn.net/detail/lqt0307/5184966)
ps:我只找到了这个下载地址，需要1分，没有分的可以找我要。

3. redmine
这个就不用说了，上官网下就行，我用的版本是2.5.2。

4. mysql
redmine使用的数据库是mysql，所以需要安装mysql。


### 安装
1. 安装railsinstaller
	如果不选择安装目录什么的，直接下一步就可以了。

2. 解压redmine-2.5.2.zip

3. 安装mysql
	这个就不多说了。

4. 安装mysql2插件
	进入Railsinstaller安装目录下的DevKit，运行DevKit目录下的msys.bat
	输入:
``` sh
gem install mysql2 -v=0.3.16 -- --with-mysql-dir="mysql安装目录"
```
命令中的"mysql安装目录"要替换掉，要不要引号都可以

5. 安装rmagick
	window下安装rmagick，需要手动安装，不能通过：gem install rmagick 进行安装。
	首先需要安装ImageMagick。安装方式可以参考[http://www.redmine.org/projects/redmine/wiki/HowTo_install_rmagick_gem_on_Windows](http://www.redmine.org/projects/redmine/wiki/HowTo_install_rmagick_gem_on_Windows)，上面讲得很详细。
	主要也是下一步、下一步就可以了，需要注意一下的是安装目录中最好不要有空格，我在这个地方被坑了好几次。

	安装完ImageMagick之后，就可以安装rmagick了。进入Railsinstaller安装目录下的DevKit，运行DevKit目录下的msys.bat。
	输入:
``` sh
gem install -- --with-opt-lib="ImageMagick安装目录/lib"  --with-opt-include="
ImageMagick安装目录/include"
```
命令中的"ImageMagick安装目录"要替换掉，要不要引号都可以。

6. 数据库准备
	6.1. 把libmysql.dll复制到“RailsInstaller安装目录\Ruby1.9.3\bin”下面。
	6.2. 创建数据库
``` sql
CREATE DATABASE db_redmine CHARACTER SET utf8;
```

	6.3. 修改redmine连接数据库文件
	redmine目录下的config/database.yml.example 改为config/database.yml
	第6行开始
``` 
production:
adapter: mysql2
database: redmine
host: localhost
username: root
password: my_password
```

7. 目录下执行:
``` sh
bundle install 
```

8. 初始化Redmine数据库环境
``` sh
bundle exec rake db:migrate RAILS_ENV=production
```

9. 生成session存储密钥
``` sh
bundle exec rake generate_secret_token
```

10. 启动Redmine
``` sh
ruby script/rails server webrick -e production
```

### 将Redmine安装为Windows服务，开机自动运行
每次都用命令去启动Redmine还是很麻烦的，把Redmine安装成windows服务，让它能够开机自启动，还是很方便的。
Ruby提供一个安装Ruby程序为服务的包：mongrel_service。安装其实很简单，只要命令行下运行gem：
``` sh
gem install mongrel_service 
```
过程中安装一些必须的其他包。
然后将RedMine使用mongrel_service安装成Windows服务：
``` sh
mongrel_rails service::install -N Redmine -c D:\WebRoot -p 3000 -e production  
```
这里，我指定服务名为RedMine，我的Redmine在D:\WebRoot，你的要修改，注意指向Redmine的根目录。监听3000端口。
然后修改启动方式为自动启动，并添加MySQL服务为其依赖服务（如果你的MySQL服务器不是本机就不用麻烦了）：
``` sh
sc config Redmine start= auto depend= MySQL  
```
注意，执行sc config系列指令，服务必须是未启动的才行，否则会出错。
将来如果想去掉这个服务，只要执行：
``` sh
mongrel_rails service::remove -N Redmine  
```
也可以使用：`sc delete Redmine` 删除服务。

以上方式只适用于xp及xp以上系统，如果你的系统是server 2003，需要用下面这种方式：[http://www.redmine.org/boards/1/topics/4123](http://www.redmine.org/boards/1/topics/4123)。
上面说得很详细了，我就不在累述。
``` sh
rake redmine:plugins:migrate RAILS_ENV=production
```
