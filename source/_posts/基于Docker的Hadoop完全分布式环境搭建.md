title: 基于Docker的Hadoop完全分布式环境搭建
date: 2015-05-19 23:55:37
tags: [java, hadoop]
comment: true
toc: true
share: true
---
### 1 配置docker环境

创建一个带有ssh的centos的docker images，命名为centos6-ssh（建议采用centos6）并且ssh能在容器启动是自启动，不是本部分的重点，不详细描述。



### 2 启动docker，配置java

<!-- more -->

2.1 启动第1步中创建的容器。

docker run -it centos6-ssh /bin/bash

2.2 安装Java

不是本文重点，不具体描述。Java的安装目录是`/opt/java8`



### 3 安装Hadoop

#### 3.1 下载Hadoop

``` sh

shell>> cd /opt

shell>> wget http://mirrors.cnnic.cn/apache/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz

shell>> tar -zxvf hadoop-2.6.0.tar.gz

```

#### 3.2 配置环境变量

``` sh

shell>> vim /etc/profile

# 输入一下内容

export HADOOP_HOME=/opt/hadoop-2.6.0/

export HADOOP_CONFIG_HOME=$HADOOP_HOME/etc/hadoop

export PATH=$PATH:$HADOOP_HOME/bin

export PATH=$PATH:$HADOOP_HOME/sbin

```

#### 3.3 配置Hadoop

下面，我们开始修改Hadoop的配置文件。主要配置core-site.xml、hdfs-site.xml、mapred-site.xml这三个文件。

``` sh

shell>> cd $HADOOP_HOME/

shell>> mkdir tmp

shell>> mkdir namenode

shell>> mkdir datanode

```

这里创建了三个目录，后续配置的时候会用到：

    1. tmp：作为Hadoop的临时目录

    2. namenode：作为NameNode的存放目录

    3. datanode：作为DataNode的存放目录

接下来进入配置文件夹，开始配置

``` sh

shell>> cd $HADOOP_CONFIG_HOME/

```

##### 3.3.1 core-site.xml配置

``` 

<?xml version="1.0" encoding="UTF-8"?>

<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!--

  Licensed under the Apache License, Version 2.0 (the "License");

  you may not use this file except in compliance with the License.

  You may obtain a copy of the License at



    http://www.apache.org/licenses/LICENSE-2.0



  Unless required by applicable law or agreed to in writing, software

  distributed under the License is distributed on an "AS IS" BASIS,

  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

  See the License for the specific language governing permissions and

  limitations under the License. See accompanying LICENSE file.

-->



<!-- Put site-specific property overrides in this file. -->



<configuration>

    <property>

            <name>hadoop.tmp.dir</name>

            <value>/opt/hadoop-2.6.0/tmp</value>

            <description>A base for other temporary directories.</description>

    </property>



    <property>

            <name>fs.default.name</name>

            <value>hdfs://master.hadoop.com:9000</value>

            <final>true</final>

            <description>The name of the default file system.  A URI whose

            scheme and authority determine the FileSystem implementation.  The

            uri's scheme determines the config property (fs.SCHEME.impl) naming

            the FileSystem implementation class.  The uri's authority is used to

            determine the host, port, etc. for a filesystem.</description>

    </property>

</configuration>

```

注意：

    * hadoop.tmp.dir配置项值即为此前命令中创建的临时目录路径。

    * fs.default.name配置为hdfs://master:9000，指向的是一个Master节点的主机（后续我们做集群配置的时候，自然会配置这个节点，先写在这里）

##### 3.3.2 hdfs-site.xml配置

```

<?xml version="1.0" encoding="UTF-8"?>

<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!--

  Licensed under the Apache License, Version 2.0 (the "License");

  you may not use this file except in compliance with the License.

  You may obtain a copy of the License at



    http://www.apache.org/licenses/LICENSE-2.0



  Unless required by applicable law or agreed to in writing, software

  distributed under the License is distributed on an "AS IS" BASIS,

  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

  See the License for the specific language governing permissions and

  limitations under the License. See accompanying LICENSE file.

-->



<!-- Put site-specific property overrides in this file. -->



<configuration>

    <property>

        <name>dfs.replication</name>

        <value>2</value>

        <final>true</final>

        <description>Default block replication.

        The actual number of replications can be specified when the file is created.

        The default is used if replication is not specified in create time.

        </description>

    </property>



    <property>

        <name>dfs.namenode.name.dir</name>

        <value>/opt/hadoop-2.6.0/namenode</value>

        <final>true</final>

    </property>



    <property>

        <name>dfs.datanode.data.dir</name>

        <value>/opt/hadoop-2.6.0/datanode</value>

        <final>true</final>

    </property>

</configuration>

```

注意：

* 我们后续搭建集群环境时，将配置一个Master节点和两个Slave节点。所以dfs.replication配置为2。

* dfs.namenode.name.dir和dfs.datanode.data.dir分别配置为之前创建的NameNode和DataNode的目录路径

##### 3.3.3 mapred-site.xml配置

``` sh

shell>> cp mapred-site.xml.template mapred-site.xml

```

```

<?xml version="1.0"?>

<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!--

  Licensed under the Apache License, Version 2.0 (the "License");

  you may not use this file except in compliance with the License.

  You may obtain a copy of the License at



    http://www.apache.org/licenses/LICENSE-2.0



  Unless required by applicable law or agreed to in writing, software

  distributed under the License is distributed on an "AS IS" BASIS,

  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

  See the License for the specific language governing permissions and

  limitations under the License. See accompanying LICENSE file.

-->



<!-- Put site-specific property overrides in this file. -->



<configuration>

    <property>

        <name>mapred.job.tracker</name>

        <value>master.hadoop.com:9001</value>

        <description>The host and port that the MapReduce job tracker runs

        at.  If "local", then jobs are run in-process as a single map

        and reduce task.

        </description>

    </property>

</configuration>

```

这里只有一个配置项`mapred.job.tracker`，我们指向master节点机器。

##### 3.3.3.4 修改JAVA_HOME环境变量

使用命令`vim hadoop-env.sh`打开配置文件，修改配置如下：

```

# The java implementation to use.

export JAVA_HOME=/opt/java8

```

#### 4 格式化namenode

这是很重要的一步，执行命令`hadoop namenode -format`

#### 5 配置SSH，生成访问秘钥

``` sh

shell>> cd ~/

shell>> ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa

shell>> cd .ssh

shell>> cat id_rsa.pub >> authorized_keys

```

注意： 这里，我的思路是直接将密钥生成后写入镜像，免得在买个容器里面再单独生成一次，还要相互拷贝公钥，比较麻烦。当然这只是学习使用，实际操作时，应该不会这么搞，因为这样所有容器的密钥都是一样的！

#### 6 创建Hadoop images

```

# 退出docker容器

shell>> exit

# commit容器

shell>> docker commit -a "zhangteng" -m "add hadoop" b7227b0d59fe02d22ad823a5c4a478ec8de3e7882488259191984995791da953 tcheung/hadoop

得到# 容器的id需要替换，可以通过docker ps -a 

```

### 3 Hadoop分布式集群搭建

重点来了！



按照 hadoop 集群的基本要求,其 中一个是 master 结点,主要是用于运行 hadoop 程序中的 namenode、secondorynamenode 和 jobtracker（新版本名字变了） 任务。用外两个结点均为 slave 结点,其中一个是用于冗余目的,如果没有冗 余,就不能称之为 hadoop 了,所以模拟 hadoop 集群至少要有 3 个结点。



我们搭建集群环境的时候，需要指定节点的hostname，以及配置hosts。hostname可以使用docker run命令的h参数直接指定，但hosts解析有点麻烦，需要每次重启之后重新配置

##### 启动master容器

``` sh

shell>> docker run -d -h master.hadoop.com  --net=none --name=master.hadoop.com tcheung/hadoop

```

##### 启动slave1容器

``` sh

shell>> docker run -d -h slave1.hadoop.com  --net=none --name=slave1.hadoop.com tcheung/hadoop



```

##### 启动slave2容器

``` sh

shell>> docker run -d -h slave2.hadoop.com  --net=none --name=slave2.hadoop.com tcheung/hadoop



```

#### 给3个容器配置ip

配置ip的方式有很多，如pipework（我使用的是这个），具体可以百度。

master	192.168.5.10

slave1	192.168.5.20

slave2 192.168.5.30

#### 修改3个容器的hosts

通过ssh分别进入3个容器，使用`vim /etc/hosts`修改各自的hosts，添加内容如下：

```

192.168.5.10    master.hadoop.com

192.168.5.20    slave1.hadoop.com

192.168.5.30    slave2.hadoop.com

```

#### 配置slaves

下面我们来配置哪些节点是slave。在较老的Hadoop版本中有一个masters文件和一个slaves文件，但新版本中只有slaves文件了。

在master节点容器中执行如下命令：

``` sh

shell>> cd $HADOOP_CONFIG_HOME/

shell>> vim slaves

```

将如下slave节点的hostname信息写入该文件： 

```

slave1.hadoop.com

slave2.hadoop.com

```

#### 启动Hadoop

在master节点上执行`start-all.sh`命令，启动Hadoop。

如果没有报错，就代表启动成功，如果有错，那就慢慢调吧。

在3个节点上执行jps命令，会得到类似的结果：

master节点

``` sh

516 ResourceManager

358 SecondaryNameNode

775 Jps

169 NameNode



```

slave1节点

``` sh

292 Jps

52 DataNode

153 NodeManager

```

slave1节点

``` sh

465 Jps

52 DataNode

153 NodeManager



```

在master节点上通过命令`hdfs dfsadmin -report`查看DataNode是否正常启动。

也可以通过在浏览器中输入`http://master.hadoop.com:50070`查看Hadoop的运行状态。























