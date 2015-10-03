title: 基于Docker的Hadoop完全分布式环境搭建
date: 2015-05-18 23:55:37
tags: [java, hadoop, hbase]
comment: true
toc: true
share: true
---
﻿﻿﻿﻿﻿
HBase需要在Hadoop的基础上运行，使用了Hadoop的HDFS，所有在搭建HBase之前，需要先搭建Hadoop的环境。
Habse的运行模式主要有两种：
1. 单机模式
2. 分布式模式
    2.1. 伪分布式
    2.2. 完全分布式
主要介绍单击模式和完全分布式。

<!-- more -->

### 1 单机模式
单机模式的HBase使用的是本地文件系统，并没有使用HDFS，这种方式可以用于实验，官方并不建议将这种方式用到生产环境中。
我选择的版本是：hbase-1.0.1（此版本是一个稳定版本）。
下载地址：http://mirrors.cnnic.cn/apache/hbase/stable/
现面开始安装说明：
#### 1.1 安装Java
不是本文重点，不做叙述。最好选择Java7或者Java8，我选择的是Java8。
#### 1.2 下载HBase并解压
``` sh
# 进入安装文件夹
shell>> cd /opt
# 下载
shell>> wget http://mirrors.cnnic.cn/apache/hbase/stable/hbase-1.0.1-bin.tar.gz
# 解压
shell>> tar -xzvf hbase-1.0.1-bin.tar.gz
```
#### 1.3 配置HBase
配置hbase-site.xml
hbase-site.xml在conf文件夹下。
``` sh
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///data/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/data/zookeeper</value>
  </property>
</configuration>
```
HBase的数据目录不需要我们去手动创建，当启HBase时，HBase会自动去创建。
#### 1.4 启动HBase
通过运行`bin/start-hbase.sh`，HBase就能启动起来。在单机模式下，HBase的所有守护进程都只运行在一个JVM下。

### 2 完全分布式模式
使用Docker来搭建分布式环境。
Docker是一种虚拟化技术，可以为我们提供虚拟化环境，启动多个Docker容器，相当于启动了多个虚拟机，各容器间是相互隔离的，基本上通过网络进行通信。
启动了3个Centos6的Docker容器来搭建HBase环境，3个容器情况如下：
|节点       |hostname         |ip          |域名                              |
|-----------|-------------------------------|----------------------|------------------------------|
|master|master.hadoop.com|192.168.5.10|master.hadoop.com|
|slave1|slave1.hadoop.com|192.168.5.20|slave1.hadoop.com|
|slave2|slave2.hadoop.com|192.168.5.30|slave1.hadoop.com|
#### 2.1 配置各Docker容器环境
##### 2.1.1 安装Java
##### 2.1.2 安装、配置SSH
在各容器中安装SSH，并生成相应的无密码登录的公钥与私钥，并叫公钥拷贝到其他容器中，确保各容器间能通过ssh无密码登录（使用公钥登录）。
``` sh
shell>> ssh-keygen -t rsa
shell>> cat id_rsa.pub >> ~/.ssh/authorized_keys
```
##### 2.1.3 修改host
通过`vim /etc/hosts`修改各容器的host，在文件中添加如下内容：
``` sh
192.168.5.10    master.hadoop.com
192.168.5.20    slave1.hadoop.com
192.168.5.30    slave2.hadoop.com
```
#### 2.2 配置master节点
##### 2.2.1 进入master节点
``` sh
ssh root@192.168.5.10
```
##### 2.2.2 下载HBase并解压
``` sh
# 进入安装文件夹
shell>> cd /opt
# 下载
shell>> wget http://mirrors.cnnic.cn/apache/hbase/stable/hbase-1.0.1-bin.tar.gz
# 解压
shell>> tar -xzvf hbase-1.0.1-bin.tar.gz
```
##### 2.2.3 配置hbase-site.xml
hbase-site.xml在conf文件夹下。
``` sh
<configuration>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.rootdir</name>
      <value>hdfs://master.hadoop/hbase</value>
    </property>


    <property>
      <name>hbase.zookeeper.quorum</name>
      <value>master.hadoop.com,slave1.hadoop.com,slave2.hadoop.com</value>
    </property>
    <property>
      <name>hbase.zookeeper.property.dataDir</name>
      <value>/opt/zookeeper</value>
    </property>
</configuration>
```
#### 2.3 配置slave1和slave2
在master中已经将HBase配置好了，HBase要求所有节点的安装目录和配置完全一致，所以，最简单的方式就是将master节点中的HBase复制到slave1和slave2
中。
可以通过`scp`达到这一目的。
在master节点中执行如下命令：
``` sh
scp /opt/hbase-1.0.1 root@192.168.5.20:/opt
scp /opt/hbase-1.0.1 root@192.168.5.30:/opt
```
#### 2.4 启动集群
在master节点中，执行`bin/start-hbase.sh`就可以启动集群。如果看到类似于下面的这些信息，就表示启动成功。
``` sh
slave1.hadoop.com: starting zookeeper, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-zookeeper-slave1.hadoop.com.out
slave2.hadoop.com: starting zookeeper, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-zookeeper-slave2.hadoop.com.out
master.hadoop.com: starting zookeeper, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-zookeeper-master.hadoop.com.out
starting master, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-master-master.hadoop.com.out
slave2.hadoop.com: starting regionserver, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-regionserver-slave2.hadoop.com.out
slave1.hadoop.com: starting regionserver, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-regionserver-slave1.hadoop.com.out
slave1.hadoop.com: starting master, logging to /opt/hbase-1.0.1/bin/../logs/hbase-root-master-slave1.hadoop.com.out
```
#### 2.5 查看各节点的进程信息
通过`jps`查看各节点运行的Java进程，如果看到如下的进程，表示HBase启动成功。
master节点
``` sh
shell>> jps
20355 Jps
20071 HQuorumPeer
20137 HMaster
```
slave1节点
``` sh
shell>> jps
15930 HRegionServer
16194 Jps
15838 HQuorumPeer
```
slave2节点
``` sh
shell>> jps
13901 Jps
13639 HQuorumPeer
13737 HRegionServer
```
#### 2.6 在浏览器中查看运行状态
在浏览器中输入：http://master.hadoop.com:16010，可查看HBase运行状态。
#### 2.7 例子演示
``` sh
建立一个表scores具有两个列族grad 和courese      
create 'scores','grade', 'course' 
查看当前HBase中具有哪些表      
list 
查看表的构造      
describe 'scores' 
加入一行数据,行名称为zkb 列族grad的列名为”” 值位5      
put 'scores','zkb','grade:','5'

给zkb这一行的数据的列族course添加一列<math,97>      
put 'scores','zkb','course:math','97'  
给zkb这一行的数据的列族course添加一列<art,87>      
put 'scores','zkb','course:art','87'  
加入一行数据,行名称为baoniu列族grad的列名为”” 值为4      
put 'scores','baoniu','grade:','4' 
给baoniu这一行的数据的列族course添加一列<math,89>     
put 'scores','baoniu','course:math','89'

给baoniu这一行的数据的列族course添加一列<art,80>      
put 'scores','baoniu','course:art','80'  
查看scores表中zkb的相关数据      
get 'scores','zkb'  
查看scores表中所有数据      
scan 'scores'  
注意：scan命令可以指定startrow,stoprow来scan多个row，例如：scan 'user_test', {COLUMNS =>'info:username',LIMIT =>10, STARTROW => 'test',STOPROW=>'test2'}

查看scores表中所有数据courses列族的所有数据      
scan 'scores',{COLUMNS => 'course'}  
删除scores表      
disable 'scores'       
drop 'scores'      
注意：先要屏蔽该表，才能对该表进行删除
```



















