title: CentOS安装Mysql
comment: true
toc: true
share: true
date: 2015-10-20 21:54:24
tags: [linux, mysql]
---
   不多说废话了，直接上教程开始安装吧。

   1. 下载Mysql

   直接上Mysql的官网下载，[http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)。
   建议下载`Linux-Generic`下面的`Compressed TAR`版本，此版本为Linux下的通用版本，是编译好之后的，不建议直接下载源码，然后进行编译安装，以为Mysql比较大，编译需要的时间比较长。
   本文使用的版本为`mysql-5.6.27-linux-glibc2.5-x86_64.tar.gz`，如下图：
   ![http://7xjta1.com1.z0.glb.clouddn.com/mysql.png](http://7xjta1.com1.z0.glb.clouddn.com/mysql.png)

<!-- more -->

   2. 安装
   
   ``` sh
   $ cp mysql-5.6.27-linux-glibc2.5-x86_64.tar.gz /opt/
   $ cd /opt/
   $ tar -zxvf mysql-5.6.27-linux-glibc2.5-x86_64.tar.gz 
   $ mv mysql-5.6.27-linux-glibc2.5-x86_64 mysql3306
   $ cd mysql3306
   ```

   在mysql目录下有一个名为`INSTALL-BINARY`的文件，这个文件是官方的安装说明，里面对如何安装mysql进行了详细的说明，大家可参考这个文件进行安装。
   本文的安装也是基于这个文件，然后结合实际情况进行改变而来。
   以下命令都是在mysql目录下进行
   ``` sh
   $ groupadd mysql
   $ useradd -r -g mysql mysql
   $ chown -R mysql .
   $ chgrp -R mysql .
   $ mkdir -p /data/db/db3306/data
   $ scripts/mysql_install_db --basedir=/opt/mysql3306 --datadir=/data/db/db3306/data --no-defaults --user=mysql
   ```

   接下来拷贝一份25上的mysql配置文件my.cnf到mysql目录下。
   my.cnf的内容在这里就不贴了，大家需要的可以找我要一份。

   ``` sh
   $ mkdir -p /data/db/db3306/log
   $ mkdir -p /data/db/db3306/binlog
   $ chown -R mysql /data/db/db3306/
   ```

   最后执行如下命令，启动一下mysql就行了：
   ``` sh
   $ bin/mysqld_safe --defaults-file=/opt/mysql3306/my.cn
   ```

   使用`ps aux | grep mysql`检查一下mysql是否启动。

   如果没有成功启动，可以使用`tail -f -n 2000 /data/db/db3306/log/mysqld.err`查看错误日志进行修改。

   3. 将Mysql添加成服务
   ``` sh
   $ cp support-files/mysql.server /etc/init.d/mysqld3306
   $ vim /etc/init.d/mysqld3306
   ```
   主要修改如下几个地方
   ``` sh
   basedir=/opt/mysql3306
   datadir=/data/db/db3306/data
   conf=/opt/mysql3306/my.cnf 
   ```
   将`$bindir/mysqld_safe --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null 2>&1`改成下面这一句:
   ```
   $bindir/mysqld_safe --defaults-file=$conf --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null 2>&
   ```

   接下来就可以使用如下`service`命令对mysql进行启动、停止、重启了。
   ``` sh
   $ service mysqld3306 start       # 启动
   $ service mysqld3306 stop        # 停止
   $ service mysqld3306 restart     # 重启
   ```



