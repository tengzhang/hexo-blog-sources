title: CentOS下Java Web环境搭建
comment: true
toc: true
share: true
date: 2015-10-13 11:22:03
tags: [linux, java]
---
CentOS下的Java Web环境，主要包括Java的安装、maven的安装、tomcat的安装，当然也包括一些svn、git之类的版本控制工具。

本文主要介绍一下与Java Web有关的软件、工具的安装。
   
本文内容主要是通过自己实践、参考网上的文章以及一些工具的官方文档总结而来，如有不足之处，欢迎批评指正。

<!-- more -->

首先说一下安装软件的一些原则(此原则为本团队原则，不是必须要这么做)，一些简单地系统工具，可以使用`yum`
直接安装，如`svn`、`git`之类的，其它一些软件需要安装到`/opt/`目录下，目的是为了将于某一个软件有关的东西
全部放到一个地方。使用`yum`安装的软件，与某一软件相关文件是到处都有的，比如可执行的程序通常在`usr/bin`或者
`/usr/local/bin`下，配置文件通常在`/etc/`下，一些临时文件通常在`/var/tmp`下。下文会讲到如何具体将软件安装到`/opt`下。

1. 开启ssh

   装完服务器之后，第一步工作当然是开启ssh。
   CentOS已经默认安装了openssh，我们不需要安装什么，只需要进行一些配置，开启ssh即可。
   1.1 配置ssh

    ``` bash 
    $ vim /etc/ssh/sshd_config
    # 修改sshd_config的如下几个地方
    Port 22
    Protocol 2
    PasswordAuthentication yes
    PermitRootLogin yes
    ```
   1.2 启动ssh
   ``` bash
   # CentOS 7以下
   $ service sshd start
   # CentOS 7
   $ systemctl start sshd
   ```
   1.3 设置开机自启动
   ``` bash 
   #CentOS 7以下
   $ chkconfig sshd on
   # CentOS 7
   $ systemctl enable sshd.service
   ```
   开启了ssh之后，以后所有的操作都可以直接在其他机器上通过ssh操作服务器了。

2. 安装版本控制工具
   ``` bash
   $ yum install svn git
   ```

3. 安装Java
   
   3.1 下载Java
   去Java官方网站下载jdk，[http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html)。
   选择Linux x64版本(如果你的机器是32位，那就选择Linux x32)，选择`.tar.gz`结尾的压缩版本，尽量不要使用`.rpm`的版本。
   本文下载的版本是`jdk-8u60-linux-x64.tar.gz`。
   
   3.2 安装
   ``` bash
   $ cp jdk-8u60-linux-x64.tar.gz /opt/     # 1. 将下载的jdk-8u60-linux-x64.tar.gz拷贝到/opt/目录下
   $ cd /opt/                              
   $ tar -zxvf jdk-8u60-linux-x64.tar.gz    # 2. 解压jdk-8u60-linux-x64.tar.gz
   $ mv jdk1.8.0_60/ java8                  # 3. 重命名
   $ vim /etc/profile                       # 4. 配置环境变量
   # 在/etc/profile中添加如下内容：
   ```
   ``` bash
   # set Java environment                                                                                                 
   export JAVA_HOME=/opt/java8
   export PATH=$PATH:$JAVA_HOME/bin
   ```
   ``` bash
   source /etc/profile                       # 5. 使配置生效 
   ```
   最后执行一下`java -version`，看看是否输出了java的版本信息，检验一下是否安装成功。

4. 安装Tomcat

   4.1 下载Tomcat
   去Tomcat官方网站下载，[http://tomcat.apache.org/download-70.cgi](http://tomcat.apache.org/download-70.cgi)。
   选择`tar.gz`版本，Linux下Tomcat不分32/64。
   本文下载的是`apache-tomcat-7.0.64.tar.gz`。

   4.2 安装
   ``` bash
   $ cp apache-tomcat-7.0.64.tar.gz /opt/   # 1. 将下载的apache-tomcat-7.0.64.tar.gz拷贝到/opt/目录下
   $ cd /opt/
   $ tar -zxvf apache-tomcat-7.0.64.tar.gz  # 2. 解压apache-tomcat-7.0.64.tar.gz
   $ mv apache-tomcat-7.0.64 tomcat7        # 3. 重命名
   ```

   4.3 启动
   ``` bash
   $ /opt/tomcat7/bin/startup.sh
   ```
   如果没有异常，并且输出当中有`Tomcat started.`字样，代表Tomcat启动成功。
   也可以通过命令`ps aux | grep java`查看Tomcat是否启动，如果有类似如下输出，表示启动。
   ![image](http://7xjta1.com1.z0.glb.clouddn.com/tomcat.png)
  
   4.4 停止
   ``` bash
   $ /opt/tomcat7/bin/shutdown.sh
   ```
   如果不好使，也可以采取暴力一点的方法，直接将java进程干掉，不过采用这种方式，所有的java程序也会停止运行。
   ```
   $ killall -9 java
   ```

5. 安装Maven

   5.1 下载Maven
   去Maven官方网站下载，[http://maven.apache.org/download.cgi#](http://maven.apache.org/download.cgi#)
   下`Binary tar.gz archive`版本。
   本文下载的版本是`apache-maven-3.3.3-bin.tar.gz`。

   5.2 安装
   ``` bash
   $ cp apache-maven-3.3.3-bin.tar.gz /opt/     # 1. 将下载的apache-maven-3.3.3-bin.tar.gz拷贝到/opt/目录下
   $ cd /opt/
   $ tar -zxvf apache-maven-3.3.3-bin.tar.gz    # 2. 解压apache-maven-3.3.3-bin.tar.gz
   $ mv apache-maven-3.3.3 maven                # 3. 重命名
   $ vim /etc/profile                           # 4. 配置环境变量
   # 在/etc/profile中添加如下内容：
   ```
   ``` bash
   # set Maven environment
   export MAVEN_HOME=/opt/maven
   export PATH=$PATH:$MAVEN_HOME/bin
   ```
   ``` bash
   $ source /etc/profile                          # 5. 使配置生效   
   ```
   最后执行以下`mvn version`，看看是否输出了maven的版本信息，如果输出了版本信息，代表安装成功。

   小结: 到这里，大家应该能发现，装Java相关的软件的步骤都差不多，下载->解压->配置环境变量->是配置生效。
   基本上经过这么几步，都可以安装好，然后进行使用了。

6. 安装nginx

   Nginx的安装和之前几个软件的安装不太一样，Nginx本身是用C写的，我们需要自己去编译它。

   6.1 安装编译环境
   ``` sh
   $ yum -y install gcc gcc-c++ automake autoconf libtool make
   ```

   6.2 下载依赖包
   主要依赖pcre、zlib、openssl
   ``` sh
   $ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
   $ wget http://zlib.net/zlib-1.2.8.tar.gz
   $ wget http://www.openssl.org/source/openssl-1.0.2d.tar.gz
   # 解压下载的依赖包
   $ tar -zxvf pcre-8.37.tar.gz 
   $ tar -zxvf zlib-1.2.8.tar.gz
   $ tar -zxvf openssl-1.0.2d.tar.gz
   ```

   6.3 下载、安装Nginx
   ``` sh
   # 下载
   $ wget http://nginx.org/download/nginx-1.8.0.tar.gz  
   # 安装
   $ ./configure --prefix=/opt/nginx --with-http_ssl_module --with-pcre=/opt/pcre-8.37 --with-zlib=/opt/zlib-1.2.8 --with-openssl=/opt/openssl-1.0.2d
   $ make
   $ make install
   ```


