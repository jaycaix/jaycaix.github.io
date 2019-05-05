---
layout: post
title: "Hadoop/Spark分布式集成环境搭建"
date:       2016-11-03 21:00:00 +0800
author: "Jay Cai"
header-style: text
catalog: true
tags:
  - Hadoop
  - Spark
  - memo
---

前言
====================================
电脑硬盘损坏，重新搭建环境，记录一下以供日后参考。

1.安装配置虚拟机
====================================

1.1. 虚拟机安装CentOS 7 minimal版本（略过）
------------------------------------

minimal版本系统没有自动安装防火墙，所以省了一些事

1.2. 配置CentOS
------------------------------------

1.2.1 修改hostname和timezone
```bash
[root@localhost ~]# hostnamectl set-hostname master
[root@localhost ~]# hostname
master
[root@localhost ~]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[root@localhost ~]# date
Wed Nov  3 10:25:56 CST 2016
```

1.2.2 添加用户组/用户
```bash
[root@localhost ~]# groupadd hadoop
[root@localhost ~]# adduser -g hadoop hadoop
[root@localhost ~]# passwd hadoop
Changing password for user hadoop.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[root@localhost ~]# groups hadoop
hadoop : hadoop

######赋予hadoop用户sudo权限
[root@localhost ~]# vi /etc/sudoers
```

1.2.3 安装telnet服务
```bash
####检查telnet有没有安装###
[root@localhost ~]# rpm -qa | grep telnet
####安装telnet###
[root@localhost ~]# yum -y install xinetd，telnet，telnet-server
###添加服务###
[root@localhost ~]# systemctl enable xinetd.service
[root@localhost ~]# systemctl enable telnet.socket
###启动服务###
[root@localhost ~]# systemctl start telnet.socket
[root@localhost ~]# systemctl start xinetd (or service xinetd start)
```

1.2.4 安装FTP Server
```bash
####putty登录到master server####
[hadoop@master ~]$ su -
Password:
Last login: Wed Nov  2 11:03:57 CST 2016 on pts/0
[root@master ~]#
###检查是否已经安装，没有则安装###
[root@master ~]# rpm -qa | grep vsftpd
[root@master ~]# yum -y install vsftpd
[root@master ~]# systemctl enable vsftpd.service
[root@master ~]# systemctl start vsftpd.service
```
至此FTP Server安装完成。如有问题，则需检查防火墙设置和selinux设置。

1.2.5 安装JDK、Scala
```bash
###如果没有安装wget 则yum -y install wget即可###
###oracle的验证比较烦 所以wget要添加下面一些参数
[root@master ~]# wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.rpm"
[root@master ~]# ls
anaconda-ks.cfg  jdk-8u112-linux-x64.rpm
[root@master ~]# rpm -ivh jdk-8u112-linux-x64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:jdk1.8.0_112-2000:1.8.0_112-fcs  ################################# [100%]
Unpacking JAR files...
		tools.jar...
		plugin.jar...
		javaws.jar...
		deploy.jar...
		rt.jar...
		jsse.jar...
		charsets.jar...
		localedata.jar...
[root@master ~]# java -version
java version "1.8.0_112"
Java(TM) SE Runtime Environment (build 1.8.0_112-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.112-b15, mixed mode)
[root@master ~]# wget
[root@master ~]# rpm -ivh scala-2.11.8.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:scala-2.11.8-0                   ################################# [100%]
[root@master ~]# scala -version
Scala code runner version 2.11.8 -- Copyright 2002-2016, LAMP/EPFL
[root@master ~]# whereis scala
scala: /usr/bin/scala /usr/share/scala /usr/share/man/man1/scala.1.gz
###配置环境变量###
[root@master ~]# vi /etc/profile
	JAVA_HOME=/usr/java/jdk1.8.0_112
	SCALA_HOME=/usr/share/scala
	PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin:$HADOOP_HOME/bin
	export JAVA_HOME SCALA_HOME PATH
[root@master ~]# source /etc/profile
```

1.2.6 修改hosts文件
```bash
###把集群中的机器添加到hosts文件###
[hadoop@master ~]$ su -
Password:
Last login: Thu Nov  3 14:21:05 CST 2016 on pts/0
[root@master ~]# vi /etc/hosts
192.168.181.128 master
192.168.181.129 slave01
192.168.181.130 slave02
```
至此，一台机器基本配置完成，克隆master作为worker节点slave01，slave02。。。

1.3. 配置master和worker之间ssh免密码登录
------------------------------------

```bash
###基于rsa算法创建ssh秘钥，完成后会生成.ssh目录###
[hadoop@master ~]$ ssh-keygen -t rsa -P ''
###把公钥添加到认证列表里###
[hadoop@master ~]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
###把公钥分发给每台slave机器###
[hadoop@master ~]$ scp ~/.ssh/id_rsa.pub hadoop@slave01:~/.ssh/id_rsa.pub.master
[hadoop@master ~]$ scp ~/.ssh/id_rsa.pub hadoop@slave02:~/.ssh/id_rsa.pub.master

###登录到slave机器，把以上生成ssh秘钥的步骤做一遍，同样要把公钥分发给其他机器###
###把其他机器分发过来的公钥添加到自己的认证列表###
[hadoop@slave01 ~]$cat ~/.ssh/id_rsa.pub.master >> ~/.ssh/authorized_keys
[hadoop@slave01 ~]$cat ~/.ssh/id_rsa.pub.slave02 >> ~/.ssh/authorized_keys

###测试ssh登陆###
[hadoop@master ~]$ ssh slave01
Warning: Permanently added the ECDSA host key for IP address '192.168.181.129' to the list of known hosts.
Last login: Thu Nov  3 15:14:08 2016 from ::ffff:192.168.181.1
[hadoop@slave01 ~]$ ssh master
Warning: Permanently added the ECDSA host key for IP address '192.168.181.128' to the list of known hosts.
Last login: Wed Nov  3 15:04:07 2016 from ::ffff:192.168.181.1
[hadoop@master ~]$ ssh slave02
Warning: Permanently added the ECDSA host key for IP address '192.168.181.130' to the list of known hosts.
Last login: Thu Nov  3 15:22:05 2016 from ::ffff:192.168.181.1
[hadoop@slave02 ~]$
####如若不通，则修改～/.ssh/authorized_keys权限为600###
```

2.安装配置Hadoop2
====================================

2.1. 下载安装hadoop-2.7.3
------------------------------------

```bash
###下载###
[hadoop@master ~]$ wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
###解压###
[hadoop@master ~]$ tar -xzvf hadoop-2.7.3.tar.gz
###添加系统变量###
[hadoop@master ~]# sudo vi /etc/profile
	HADOOP_HOME=/home/hadoop/hadoop-2.7.3
	HADOOP_CONF_DIR=/home/hadoop/hadoop-2.7.3/etc/hadoop
	PATH=$PATH:$HADOOP_HOME/bin
	export HADOOP_HOME HADOOP_CONF_DIR PATH
[hadoop@master ~]# source /etc/profile
```

2.2. 修改hadoop配置文件
------------------------------------

hadoop的配置文件均在$HADOOP_CONF_DIR目录下，下面依次修改（均为基本配置）

2.2.1 hadoop-env.sh
```bash
export JAVA_HOME=/usr/java/jdk1.8.0_112
```

2.2.1 core-site.xml
```xml
<configuration>
	<!-- 指定namenode的地址 -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://master:9000</value>
	</property>
	<!-- 指定缓冲区大小128M（默认值） -->
	<property>
		<name>io.file.buffer.size</name>
		<value>131072</value>
	</property>
	<!-- 指定hadoop运行时临时文件夹路径，事先建好 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/home/hadoop/hadooptmp</value>
	</property>
</configuration>
```

2.2.2 hdfs-site.xml
```xml
<configuration>
	<!-- 设置namenode(主节点)的地址及端口 -->
	<property>
		<name>dfs.namenode.http-address</name>
		<value>master:50070</value>
	</property>

	<!-- 设置secondarynamenode(从节点)的地址及端口 -->
	<property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>slave1:50090</value>
	</property>

	<!-- 设置namenode存储路径，事先建好 -->
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:/hdfs/namenode</value>
	</property>

	<!-- 设置hdfs副本数量 -->
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
	<!-- 设置datanode存储路径，事先建好-->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:/hdfs/datanode</value>
	</property>
	<!-- 设置webhdfs为true -->
	<property>
		<name>dfs.webhdfs.enabled</name>
		<value>true</value>
	</property>
</configuration>
```

2.2.3 yarn-site.xml
```xml
<configuration>
	<!-- 设置 resourcemanager 在哪个节点，配置此项就可以省略yarn.resourcemanager.*.address以使用默认端口-->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>master</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		 <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
		 <value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>
</configuration>
```

2.2.4 mapred-site.xml
```xml
<configuration>
	<!-- 设置MR运行框架为YARN -->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<!-- 设置jobhistory server的地址及端口-->
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>master:10020</value>
	</property>
	<!-- 设置jobhistory server的web UI地址及端口-->
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>master:19888</value>
	</property>
</configuration>
```

2.2.5 masters文件
```bash
master
```

2.2.6 slaves文件
```bash
slave01
slave02
```

2.3 分发hadoop到slave机器
------------------------------------

分发机器比较多的情况可以用pssh来做批量分发，需要安装pssh这个工具，在此用了pssh工具提供的pscp命令一次性给所有机器分发文件

2.3.1 用pscp分发整个hadoop安装目录到slave机器
```bash
###创建一个文件，记录需要分发文件给哪些机器###
[hadoop@master ~]$ vi host.txt
	hadoop@slave01
	hadoop@slave02
###批量分发hadoop安装目录到slave###
[hadoop@master ~]$ pscp -h hosts.txt -r  ~/hadoop-2.7.3 ~/
[1] 08:26:46 [SUCCESS] hadoop@slave02
[2] 08:26:46 [SUCCESS] hadoop@slave01
```

2.3.2 分发修改后的/etc/profile到slave机器启动hadoop

2.4 启动集群
------------------------------------

第一次启动集群，需要格式化namenode
```bash
[hadoop@master hadoop-2.7.3]$ ./bin/hdfs namenode -format
```

2.4.1 启动
```bash
[hadoop@master hadoop-2.7.3]$ ./sbin/start-all.sh
```

2.4.2 查看进程
```xml
[hadoop@master hadoop-2.7.3]$ jps
4401 Jps
2804 ResourceManager
2533 NameNode

[hadoop@slave01 ~]$ jps
2469 NodeManager
2365 SecondaryNameNode
4637 Jps
2287 DataNode

[hadoop@slave02 ~]$ jps
2376 NodeManager
2265 DataNode
3609 Jps
```

2.4.3 web页面查看状态

http://master_ip:8088/
http://master_ip:50070/


3.安装配置spark2
====================================

3.1 下载spark-2.0.1
------------------------------------

```bash
###下载###
[hadoop@master ~]$ wget http://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-2.0.1/spark-2.0.1-bin-hadoop2.7.tgz
###解压###
[hadoop@master ~]$ tar -xzvf spark-2.0.1-bin-hadoop2.7.tgz
```

3.2 修改spark配置文件
------------------------------------

3.2.1 spark-env.sh
```bash
[hadoop@master conf]$ pwd
/home/hadoop/spark-2.0.1-bin-hadoop2.7/conf
[hadoop@master conf]$ cp spark-env.sh.template spark-env.sh
[hadoop@master conf]$ vi spark-env.sh
	SPARK_MASTER_HOST=master
[hadoop@master conf]$
```

3.2.2 slaves文件
```bash
[hadoop@master conf]$ vi slaves
	slave01
	slave02
[hadoop@master conf]$
```

3.3 分发spark到slave机器
------------------------------------

```bash
###批量分发spark安装目录到slave###
[hadoop@master ~]$ pscp -h hosts.txt -r  ~/spark-2.0.1-bin-hadoop2.7 ~/
[1] 09:13:57 [SUCCESS] hadoop@slave02
[2] 09:13:57 [SUCCESS] hadoop@slave01
```

3.4 启动spark
------------------------------------

3.4.1 启动
```bash
[hadoop@master spark-2.0.1-bin-hadoop2.7]$ ./sbin/start-all.sh
```

3.4.2 查看进程
```bash
[hadoop@master spark-2.0.1-bin-hadoop2.7]$ jps
2804 ResourceManager
2533 NameNode
4488 Jps
3083 Master

[hadoop@slave01 ~]$ jps
2610 Worker
2469 NodeManager
2365 SecondaryNameNode
4813 Jps
2287 DataNode

[hadoop@slave02 ~]$ jps
2517 Worker
2376 NodeManager
3704 Jps
2265 DataNode
```

3.4.3 web页面查看

http://master_ip:8080/
http://master_ip:4040/ 需要在运行job时才能看到

至此，一套简单可用的分布式集成环境搭建完毕。