title: 基于docker的gitlab搭建
comment: true
toc: true
share: true
date: 2015-05-03 14:21:48
tags: [linux, docker, gitlab]
---

﻿﻿﻿﻿﻿﻿﻿### 一、准备docker容器环境
#### 1.&nbsp;下载docker ubuntu镜像

``` sh
shell>> docker pull ubuntu
```
#### 2.&nbsp;启动ubuntu镜像并更新

``` sh
shell>> docker run -it docker.io/ubuntu
# 更新操作在ubuntu容器中进行
shell>> apt-get -y update
```

<!-- more -->

#### 3.&nbsp;安装ssh

``` sh
shell>> apt-get install openssh-server vim
# 配置ssh
shell>> vim /etc/ssh/sshd_config
# 启动ssh
shell>> service ssh start
# 退出容器
shell>> exit
```
#### 4.&nbsp;commit docker容器

``` sh
shell>> docker commit -a "zhangteng" -m "add ssh" 876f90b7a5769c04f3d9975369b3a1148e6c951d1f4f4217728b072b6ae6a50b ubuntu-ssh
ps: "876f90b7a5769c04f3d9975369b3a1148e6c951d1f4f4217728b072b6ae6a50b"为刚刚安装了ssh的ubuntu容器的id，需要替换，id可通过ps -a查到
```
#### 5.&nbsp;build docker容器
``` sh
shell>> mkdir ubuntu_ssh
shell>> cd ubuntu_ssh
shell>> vim Dockerfile
```
在Dockerfile文件中输入如下内容:
``` sh
# Add ssh to ubuntu
# author: zhangteng
# VERSION 1.0

FROM ubuntu-ssh:latest
MAINTAINER zhangteng
 
# change root passwd
RUN echo 'root:123456' | chpasswd
 
# expose ssh port
EXPOSE 22
 
RUN mkdir /var/run/sshd                                                                                                                  
 
# add run.sh
ADD run.sh /run.sh
RUN chmod 755 /run.sh

CMD ["/run.sh"]
```
``` sh
shell>> vim run.sh
```
 在run.sh文件中输入如下内容：
``` sh
#!/bin/bash
 
# start sshd, let this at last
/usr/sbin/sshd -D 
```
build容器
``` sh
docker build -t ubuntu-ssh:v1 .
```
#### 6.&nbsp;启动容器并进入容器
``` sh
shell>> docker run -d -p 49170:22 --name=ubuntu-ssh ubuntu-ssh:v1
shell>> ssh root@127.0.0.1 -p 49170
```
### 二、安装gtilab
在完成上一步，进入docker容器之后就可以开始安装gitlab了。
注意：以下操作都必须在**docker容器**中进行，不在容器中的操作会有说明，没有说明的都在容器中操作。
以下内容参考[gitlab官方安装说明](http://https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md)，根据我们这边要求进行了一些说明。
#### 1.&nbsp;Packages/Dependencies
1.1&nbsp;安装需要的依赖（用于编译Ruby和一些Ruby gems需要的原生依赖）
``` sh
shell>> sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server redis-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake libkrb5-dev nodejs
```
1.2&nbsp;检查git的版本是否正确
``` sh
# 安装Git
shell>> sudo apt-get install -y git-core

# 确保Git的版本是1.7.10或者更高
shell>> git --version
```
如果git的版本太久，可以移除它并用源码安装的方式安装一个高版本。
``` sh
# 移除旧版本的Git
shell>> sudo apt-get remove git-core

# 安装依赖
shell>> sudo apt-get install -y libcurl4-openssl-dev libexpat1-dev gettext libz-dev libssl-dev build-essential

# 下载源码、编译
shell>> cd /tmp
shell>> curl -L --progress https://www.kernel.org/pub/software/scm/git/git-2.1.2.tar.gz | tar xz
shell>> cd git-2.1.2/
shell>> ./configure
shell>> make prefix=/usr/local all

# 安装到/usr/local/bin
shell>> sudo make prefix=/usr/local install

# When editing config/gitlab.yml (Step 5), change the git -> bin_path to /usr/local/bin/git
# 当在第6步编辑config/gitlab.yml的时候，需要编辑bin_path=/usr/local/bin/git
``` 
1.3&nbsp;安装postfix
做邮件服务器用，但是现在没有调通
``` sh
shell>> sudo apt-get install -y postfix
```
#### 2.&nbsp;安装Ruby
2.1&nbsp;移除旧版本的Ruby
``` sh
shell>> sudo apt-get remove ruby1.8
```
2.2&nbsp;下载Ruby源码，编译并安装
``` sh
shell>> mkdir /tmp/ruby && cd /tmp/ruby
shell>> curl -L --progress http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.6.tar.gz | tar xz
shell>> cd ruby-2.1.6
shell>> ./configure --disable-install-rdoc
shell>> make
shell>> sudo make install
```
2.3&nbsp;更改Gem源
由于国内网络原因（你懂的），导致 rubygems.org 存放在 Amazon S3 上面的资源文件间歇性连接失败。所以你会与遇到 gem install rack 或 bundle install 的时候半天没有响应，具体可以用 gem install rails -V 来查看执行过程。
详见[http://ruby.taobao.org/](http://ruby.taobao.org/)。
``` sh
shell>> gem sources --remove https://rubygems.org/
shell>> gem sources -a https://ruby.taobao.org/
shell>> gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org
# 请确保只有 ruby.taobao.org
```
2.4&nbsp;安装Bundler Gem
``` sh
shell>> sudo gem install bundler --no-ri --no-rdoc
```
#### 3.&nbsp;添加系统用户
``` sh
shell>> sudo adduser --disabled-login --gecos 'GitLab' git
```
#### 4.&nbsp;安装数据库(MySql)
我们使用的数据库是宿主主机的数据库，这里安装只是为了用于后面安装Gem，不需要启动这个数据库。
数据库可以选择Mysql或者PostgreSQL，我们使用的是Mysql。
``` sh
# 安装Mysql
sudo apt-get install -y mysql-client libmysqlclient-dev

# Ensure you have MySQL version 5.5.14 or later
mysql --version

# 输入mysql的密码
# 再次输入mysql的密码
```
安装完数据库之后，还需要在**宿主主机**的Mysql中创建gitlab需要用的账户和数据库。
以下操作在宿主主机中执行。
``` sh
# 登录Mysql
shell>> mysql -h127.0.0.1 -uroot -p

# 输入mysql密码

# 创建git账户
mysql> CREATE USER 'git'@'localhost' IDENTIFIED BY 'gitlab';

# 创建Gitlab生产环境数据库
mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, LOCK TABLES ON `gitlabhq_production`.* TO 'git'@'localhost';

# 为git添加远程访问能力
mysql> GRANT ALL PRIVILEGES ON *.* TO git@"192.168.5.%" IDENTIFIED BY "git"; 
mysql> GRANT ALL PRIVILEGES ON *.* TO git@"172.%.%.%" IDENTIFIED BY "git"; 
# 更改git只能在192.168.5.*这个ip段访问
mysql> use mysql;
mysql> update user set Host="192.168.5.%" where user="git" and Host="%";
mysql> flush privileges;
mysql> \q;
```
#### 5.&nbsp;安装Redis
以下操作在docker容器中进行。
``` sh
shell>> sudo apt-get install redis-server

# 配置redis使用socket访问
shell>> sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig

# 将redis监听端口设为0，取消其对TCP请求的监听
shell>> sed 's/^port .*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf
echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf
echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf


# 创建socket需要使用的目录
shell>> mkdir /var/run/redis
shell>> chown redis:redis /var/run/redis
shell>> chmod 755 /var/run/redis
# Persist the directory which contains the socket, if applicable
if [ -d /etc/tmpfiles.d ]; then
  echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf
fi

# 重启以激活修改
shell>> sudo service redis-server restart

# 将git用户添加到redis组
shell>> sudo usermod -aG redis git
```
#### 6.&nbsp;安装Gitlab
6.1&nbsp;克隆Gitlab源码
``` sh
shell>> cd /home/git
shell>> sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 7-10-stable gitlab
ps: 我有下载好的，如果下载速度太慢，可以找我要，我也会传到gitlab上
```
6.2&nbsp;配置Gitlab
``` sh
shell>> cd /home/git/gitlab

# 复制一份gitlab的配置文件
shell>> sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml

# 根据配置文件头部的说明进行修改，可以参考我修改之后的文件，在gitlab中
shell>> sudo -u git -H vim config/gitlab.yml
# 这需要修改production这一块就可以了，列出几个需要修改的关键地方
host: gitlab.b-m.net
port: 443
https: true
email_from: gitlab@boman.net
repos_path: /data/gitlab-data/repositories/

# 确保Gitlab对log/和tmp/这两个目录有写的权限
shell>> sudo chown -R git log/
shell>> sudo chown -R git tmp/
shell>> sudo chmod -R u+rwX,go-w log/
shell>> sudo chmod -R u+rwX tmp/

# 创建satellites文件夹
shell>> sudo -u git -H mkdir /home/git/gitlab-satellites
shell>> sudo chmod u+rwx,g=rx,o-rwx /home/git/gitlab-satellites

# 确保Gitlab对tmp/pids/和tmp/sockets/两个目录有写权限
shell>> sudo chmod -R u+rwX tmp/pids/
shell>> sudo chmod -R u+rwX tmp/sockets/

# 确保Gitlab对public/uploads/目录有写权限
shell>> sudo chmod -R u+rwX  public/uploads

# 复制一份Unicorn配置文件
shell>> sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb

# 查找cpu的核数
nproc

# 如果你希望有一个高负荷的实例，可以开启集群模式
# 例如. 如果内存为2GB，设置workers为3
# workers的值至少是cpu的核数
shell>> sudo -u git -H vim config/unicorn.rb
列出需要修改的关键地方
worker_processes 4
listen "/home/git/gitlab/tmp/sockets/gitlab.socket", :backlog => 1024
listen "8080", :tcp_nopush => true

# Copy the example Rack attack config
shell>> sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

# Configure Git global settings for git user, useful when editing via web
# Edit user.email according to what is set in gitlab.yml
shell>> sudo -u git -H git config --global user.name "GitLab"
shell>> sudo -u git -H git config --global user.email "gitlab@b-m.net"
shell>> sudo -u git -H git config --global core.autocrlf input

# 配置Redis连接设置
shell>> sudo -u git -H cp config/resque.yml.example config/resque.yml

# 如果没有使用默认方式安装Redis，需要修改Redis的socket path
shell>> sudo -u git -H editor config/resque.yml
```
6.3&nbsp;配置Gitlab的数据库设置
``` sh
shell>> sudo -u git cp config/database.yml.mysql config/database.yml
shell>> sudo -u git -H vim config/database.yml
只需要修改production部分即可
production:
  adapter: mysql2
  encoding: utf8
  collation: utf8_general_ci
  reconnect: false
  database: gitlabhq_production
  pool: 10
  username: git
  password: "gitlab"
  host: 192.168.5.1
  socket: /alidata1/data/db3306/mysql.sock
```
6.4&nbsp;安装Gems
6.4.1&nbsp;修改Gem源
``` sh
shell>> sudo -u git -H vim Gemfile
修改第一行
# source "https://rubygems.org"
source 'https://ruby.taobao.org/'
```
6.4.2&nbsp;安装
``` sh
shell>> sudo -u git -H bundle install --deployment --without development test postgres aws
```
6.5&nbsp;安装Gitlab Shell
``` sh
shell>> mkdir -p /data/gitlab-data/repositories/
shell>> chown -R git:git /data/gitlab-data/
shell>> sudo -u git -H bundle exec rake gitlab:shell:install[v2.6.2] REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production
# 修改配置
shell>> sudo -u git -H vim /home/git/gitlab-shell/config.yml
# 修改后的内容如下，也可参考我上传的文件
user: git
gitlab_url: https://gitlab.b-m.net/
http_settings:
  self_signed_cert: true
repos_path: "/data/gitlab-data/repositories/"
auth_file: "/data/gitlab-data/.ssh/authorized_keys"
redis:
  bin: "/usr/bin/redis-cli"
  namespace: resque:gitlab
  socket: "/var/run/redis/redis.sock"
log_level: INFO
audit_usernames: false
```
6.6&nbsp;初始化数据库
``` sh
shell>> sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
# 输入"yes"去创建数据库表
# 初始化完成之后可以看见"Administrator account created:"
```
可以使用下面的语句设置管理员的密码，如果不设置将使用默认密码。
``` sh
shell>> sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production GITLAB_ROOT_PASSWORD=yourpassword
```
6.7&nbsp;安装服务脚本
``` sh
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
# 设置开机自启动
sudo update-rc.d gitlab defaults 21
```
6.8&nbsp;安装日志管理工具
``` sh
shell>> sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
```
6.9&nbsp;检查程序状态
``` sh
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
```
6.10&nbsp;预编译
``` sh
sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
```
6.11&nbsp;启动Gitlab
``` sh
sudo service gitlab start
```
7.&nbsp;退出docker容器
``` sh
shell>> exit;
```
**到这里，gitlab已经可以投入使用了，后面的操作只是对docker进行一些更多的操作，保存镜像，配置静态ip。**
### 三、将Gitlab容器做成docker镜像，配置静态ip
#### 1.&nbsp;commit docker容器
这里commit的是刚刚安装了Gitlab的容器。
``` sh
docker commit -a "zhangteng" -m"add gitlab" 26d8f1393450e6743e2f6becfb2ca4574e04cbb101234bb95f04d157fa9af6c2 ubuntu-gitlab
ps: "26d8f1393450e6743e2f6becfb2ca4574e04cbb101234bb95f04d157fa9af6c2"为刚刚安装了Gitlab的ubuntu-ssh容器的id，需要替换，id可通过ps -a查到
```
#### 2.&nbsp;build docker容器
``` sh
shell>> mkdir ubuntu_gitlab
shell>> cd ubuntu_gitlab
shell>> vim Dockerfile
```
在Dockerfile文件中输入如下内容:
```
# Add gitlab to ubuntu
# author: zhangteng
# VERSION 1.0

FROM ubuntu-gitlab:latest
MAINTAINER zhangteng

# expose port 80
EXPOSE 80

# expose port 443
EXPOSE 443

# add run.sh
ADD run.sh /run.sh
RUN chmod 755 /run.sh

CMD ["/run.sh"]

```
``` sh
shell>> vim run.sh
```
在run.sh文件中输入如下内容：
``` sh
#!/bin/bash

# start redis
/etc/init.d/redis-server stop >/dev/null 2>&1
/etc/init.d/redis-server start >/dev/null 2>&1
# start gitlab
/etc/init.d/gitlab stop >/dev/null 2>&1 
/etc/init.d/gitlab start >/dev/null 2>&1

# start sshd, let this at last
/usr/sbin/sshd -D >/dev/null 2>&1
```
最后，build容器
``` sh
docker build -t ubuntu-gitlab:v1 .
```
#### 3.&nbsp;启动docker容器
``` sh
docker run -d -v /alidata1/docker-gitlab:/data/gitlab-data --net=none --name=gitlabv1 ubuntu-gitlab:v1
-v参数需要根据实际情况修改
/alidata1/docker-gitlab文件夹需要先创建好
```
#### 4.&nbsp;为docker容器配置静态ip
可以使用我提供的脚本docker_static_ip.sh为docker设置静态ip
使用方法如下为：`sh  docker_static_ip.sh 容器id/容器name ip`
容器id可以通过`docker ps`得到
``` sh
sh docker_static_ip.sh 69ead0a0e1ba266f97232f4f51940e9dd2da21952ce8517b99c699b66ad10a24 192.168.5.5
```
#### 5.&nbsp;进入docker容器
由于第3步启动docker容器的时候，docker容器还没有ip，在第4步的时候才设置，Gitlab在启动的时候会去连接Mysql，这个时候是连不上的，所以启动不起来，需要进入手动启动
``` sh
shell>> ssh root@192.168.5.5
# 输入docker容器的root密码
shell>> service gitlab restart
shell>> chown -R git:git /data/gitlab-data
shell>> exit
```
#### 6.&nbsp;在宿主主机中配置反向代理
配置反向代理到docker容器中，具体配置参考gitlab.b-m.net.conf文件。
我们使用的是https，但是没有证书，所以需要自己生成一个，可以用下面的命令：
``` sh
shell>> openssl req -newkey rsa:2048 -x509 -nodes -days 3560 -out gitlab.crt -keyout gitlab.key
``` 
#### 7.&nbsp;访问
在浏览器中输入地址访问即可
### 四、使用中的一些问题
#### 1.&nbsp;设置git忽略对https的检测
由于我们使用的是自签名证书，git会对https进行检测，所以需要设置git对它忽略
``` sh
shell>> git config --global http.sslVerify "false"
```
#### 2.&nbsp;让git记住用户名和密码
编辑工程目录下的.git/config，在最后增加
``` sh
[credential]
helper = store
```