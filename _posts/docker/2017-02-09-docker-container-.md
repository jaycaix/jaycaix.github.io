---
layout:     post
title:      "Docker常用命令"
date:       2017-02-09 11:00:00 +0800
author:     "Jay Cai"
header-style: text
catalog: true
tags:
    - Docker
---

## 1\. 创建docker image

### 1\.1 创建docker file
```bash
$ mkdir mydockerbuild
$ cd mydockerbuild
$ touch Dockerfile
$ vi Dockerfile
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```
注：以上是以docker/whalesay镜像为基础，增加的个性化定制内容。

### 1\.2 依据dockerfile创建image
`$ docker build -t docker-whale .`

### 1\.3 运行刚刚创建的docker镜像
```bash
$ docker images
$ docker run docker-whale
```
以上仅为功能测试，更为详细的创建dockerfile的方法参见本文最后一个章节。

## 2\. 创建一个docker hub account，上传镜像
```bash
Docker Pull Command:
docker pull watermelonbig/myfirstrepo
$ docker images;
REPOSITORY TAG IMAGE ID CREATED SIZE
docker-whale latest d780e3732847 About an hour ago 256.2 MB
hello-world latest c54a2cc56cbb 9 weeks ago 1.848 kB
docker/whalesay latest 6b362a9f73eb 15 months ago 247 MB
```
重命名：
```bash
$ docker tag 7d9495d03763 maryatdocker/docker-whale:latest
$ docker login --username=watermelonbig --email=watermelonbig@163.com
Flag --email has been deprecated, will be removed in 1.13.
Password:
Login Succeeded
$ docker push watermelonbig/docker-whale
```
删除本地的docker镜像，然后直接使用线上docker hub中的镜像资源：
```bash
$ docker rmi -f d780e3732847
$ docker run watermelonbig/docker-whal
```

## 3\. docker常用命令
1. 运行一个交互式的终端窗口：
`$ docker run -t -i ubuntu /bin/bash`
2. 以后台方式运行一个docker容器：
```bash
$ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
35ee67330e29f56fc19c8794ede22e3d3bd8198155e1fc2b428dff2090615019
```
3. 查看在运行的容器：
```bash
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
35ee67330e29 ubuntu "/bin/sh -c 'while tr" 27 seconds ago Up 27 seconds desperate_engelbart
```
4. 查看容器的输出内容：
`$ docker logs desperate_engelbart`
5. 停止一个容器：
`$ docker stop desperate_engelbart`
6. 以后台方式运行一个web app：
`$ docker run -d -P training/webapp python app.py`
注：-P参数，作用是提供端口自动映射。
`$ docker run -d -p 80:5000 training/webapp python app.py`
注：-p参数，作用是提供端口手动映射。
7. 为窗口指定新的名称：
`$ docker run -d -P --name web training/webapp python app.py`
8. 查看容器和端口：
`$ docker ps -l`
9. 直接查看容器的端口映射结果：
`$ docker port nostalgic_morse 5000`
10. 查看容器的持续输出：
`$ docker logs -f nostalgic_morse`
11. 查看容器内运行中的进程：
`$ docker top nostalgic_morse`
12. 查看容器的详细信息：
`$ docker inspect nostalgic_morse`
13. 启动一个之前的容器：
`$ docker start nostalgic_morse`

14. 重启一个容器：
`$ docker restart nostalgic_morse`
15. 怎样从线上获取镜像：
```bash
$ docker search centos
NAME DESCRIPTION STARS OFFICIAL AUTOMATED
centos The official build of CentOS. 2606 [OK]
jdeathe/centos-ssh CentOS-6 6.8 x86_64 / CentOS-7 7.2.1511 x8... 28 [OK]
jdeathe/centos-ssh-apache-php CentOS-6 6.8 x86_64 / Apache / PHP / PHP M... 18 [OK]
nimmis/java-centos This is docker images of CentOS 7 with dif... 14 [OK]
million12/centos-supervisor Base CentOS-7 with supervisord launcher, h... 12 [OK]
```
官方可信镜像的命名方式较其它个人分享的镜像有所不同。
```bash
$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
3d8673bd162a: Downloading [==========================================> ] 60.01 MB/70.58 MB
```
注：不指定版本的情况下，按下载最新版本处理。
下载指定版本的镜像：
```bash
$ docker pull centos:6.8
6.8: Pulling from library/centos
b6d7b2ebc0a7: Downloading [====================================> ] 50.82 MB/68.75 MB
```
注：可以登录dockerhub查找centos镜像，查看其有哪些版本的标签。

## 4\. docker的数据卷管理

为容器添加一个数据卷并挂载在本地/webapp目录：
```bash
$ docker run -d -P --name web -v /webapp training/webapp python app.py
2e48371c6ef54e055a97cc6782e700148dd8e6ce0ff6d51d71c911c0aed3ec70
$ docker exec -it web bash
root@2e48371c6ef5:/opt/webapp# df -h
/dev/mapper/centos-home 767G 904M 766G 1% /webapp
docker volume ls
```
查看容器的数据卷挂载
```bash
$ docker inspect web
"Mounts": [
{
"Name": "48a19023f78bc5b677496f4109773eba51b4e1c8aee2330b97e6f5da4c32446d",
"Source": "/var/lib/docker/volumes/48a19023f78bc5b677496f4109773eba51b4e1c8aee2330b97e6f5da4c32446d/_data",
"Destination": "/webapp",
"Driver": "local",
"Mode": "",
"RW": true,
 "Propagation": ""
 }
 ],
```
使用共享存储为docker划分数据卷：
```bash
$ docker volume create -d flocker --name my-named-volume -o size=20GB
$ docker run -d -P -v my-named-volume:/opt/webapp --name web training/webapp python app.py
# create a new volume mounted to your host machine
docker run -v /host/logs:/container/logs ubuntu echo momma
```
支持基于文件的挂载：
`$ docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash`
支持创建一个共享的数据卷，然后在多个容器间挂载：
```bash
$ docker create -v /dbdata --name dbstore training/postgres /bin/true
$ docker run -d --volumes-from dbstore --name db1 training/postgres
$ docker run -d --volumes-from dbstore --name db2 training/postgres
```
查看docker使用的存储驱动类型：
`$ docker info`

如果需要更改docker进程使用的驱动：
`$ dockerd --storage-driver=devicemapper &`

## 5\. docker的存储配置方案
详细内容可以参考下面的链接。
[https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/)

### 5\.1 docker默认使用的存储驱动是devicemapper，运行在loop-lvm模式
```bash
[root@localhost lib]# docker info
Containers: 23
Running: 0
Paused: 0
Stopped: 23
Images: 7
Server Version: 1.12.1
Storage Driver: devicemapper
Pool Name: docker-253:0-135494546-pool
 Pool Blocksize: 65.54 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
```
这就是说docker的数据文件和元数据文件都是保存在/var/lib/docker/devicemapper/devicemapper目录下的。
因为devicemapper的性能表现差，所以官方推荐**在生产环境中使用direct-lvm模式**。
direct-lvm模式是直接使用块设备创建docker使用的精简配置存储池。精简配置存储池，是一种存储上的概念，即分配给用户的存储空间不会被直接整块空间的分配出去，而是按用户实际使用动态得扩展，但不能超出用户可以使用的存储空间上限。

### 5\.2 创建direct-lvm模式所需的LVM逻辑卷
系统中需要有空闲的存储空间，才可以创建出所需的LVM逻辑卷。
使用lvdisplay查看系统现状，使用lvcreate创建direct-lvm所需的数据卷和元数据卷。
以下为创建结果，thinpool保存docker数据，thinpoolmeta保存元数据，data-volume是设计为直接以docker volume方式提供给docker实例使用的数据磁盘。
```bash
$ lvcreate --wipesignatures y -n thinpool centos -L 200g
$ lvcreate --wipesignatures y -n thinpoolmeta centos -L 2g
$ lvcreate --wipesignatures y -n data-volume centos -L 560g
# lvs
LV VG Attr LSize Pool Origin Data% Meta% Move Log Cpy%Sync Convert
data-volume centos -wi-a----- 560.00g
root centos -wi-ao---- 50.00g
swap centos -wi-ao---- 17.69g
 thinpool centos -wi-a----- 200.00g
 thinpoolmeta centos -wi-a----- 2.00g
```

### 5\.3 Convert the pool to a thin pool创建一个精简配置存储池
```bash
[root@localhost lib]# lvconvert -y --zero n -c 512K --thinpool centos/thinpool --poolmetadata centos/thinpoolmeta
WARNING: Converting logical volume centos/thinpool and centos/thinpoolmeta to pool's data and metadata volumes.
THIS WILL DESTROY CONTENT OF LOGICAL VOLUME (filesystem etc.)
Converted centos/thinpool to thin pool.
```
创建一个lvm配置文件：
```bash
# vi /etc/lvm/profile/docker-thinpool.profile
activation {
thin_pool_autoextend_threshold=80
thin_pool_autoextend_percent=20
}
# lvchange --metadataprofile docker-thinpool centos/thinpool
Logical volume "thinpool" changed.
[root@localhost lib]#
```
确认thinpool处于monitored状态：
```bash
# lvs -o+seg_monitor
LV VG Attr LSize Pool Origin Data% Meta% Move Log Cpy%Sync Convert Monitor
docker-volume centos -wi-a----- 560.00g
root centos -wi-ao---- 50.00g
swap centos -wi-ao---- 17.69g
thinpool centos twi-a-t--- 200.00g 0.00 0.01 monitored
```

### 5\.4 配置docker后台进程的devicemapper启动参数
```bash
systemctl stop docker
rm -rf /var/lib/docker/*
cd /etc/docker
vi daemon.json
{
"storage-driver": "devicemapper",
"storage-opts": [
"dm.thinpooldev=/dev/mapper/centos-thinpool",
 "dm.use_deferred_removal=true"
 ]
 }
 systemctl daemon-reload
 systemctl start docker
 docker info
 Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
 Images: 0
 Server Version: 1.12.1
 Storage Driver: devicemapper
 Pool Name: centos-thinpool
 Pool Blocksize: 524.3 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file:
 Metadata file:
 Data Space Used: 19.92 MB
 Data Space Total: 214.7 GB
 Data Space Available: 214.7 GB
 Metadata Space Used: 274.4 kB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.147 GB
 Thin Pool Minimum Free Space: 21.47 GB
```
### 5\.5 把数据卷data-volume挂载为/var/lib/docker
因为我们希望docker容器中的数据磁盘直接使用宿主机的数据分区，避开docker存储驱动的各种性能瓶颈，所以在宿主机上创建的专用数据分区data-volume。
停服务：
`systemctl stop docker`
先把/var/lib/docker目录中的内容备份到别处：
```bash
mkdir -p /tmp/docker-data
mv /var/lib/docker/* /tmp/docker-data
```
手动挂载并在/etc/fstab中增加相关配置内容：
```bash
/dev/mapper/centos-data--volume /var/lib/docker xfs defaults 0 0
# df -h
Filesystem Size Used Avail Use% Mounted on
/dev/mapper/centos-root 50G 2.3G 48G 5% /
devtmpfs 18G 0 18G 0% /dev
tmpfs 18G 0 18G 0% /dev/shm
tmpfs 18G 25M 18G 1% /run
tmpfs 18G 0 18G 0% /sys/fs/cgroup
 /dev/sda2 497M 157M 341M 32% /boot
 /dev/sda1 200M 9.5M 191M 5% /boot/efi
 tmpfs 3.6G 0 3.6G 0% /run/user/0
 /dev/mapper/centos-data--volume 560G 521M 560G 1% /var/lib/docker
```
把之前备份的数据放回原地：
`mv   /tmp/docker-data/*  /var/lib/docker/`
启动docker后台服务：
`systemctl start docker`

### 5\.6 关于逻辑卷空间使用率的监控
```bash
# lvs
LV VG Attr LSize Pool Origin Data% Meta% Move Log Cpy%Sync Convert
docker-volume centos -wi-a----- 560.00g
root centos -wi-ao---- 50.00g
swap centos -wi-ao---- 17.69g
thinpool centos twi-a-t--- 200.00g 0.01 0.01
```

### 5\.7查看centos-thinpool逻辑卷的使用日志
```bash
# journalctl -fu dm-event.service
-- Logs begin at Wed 2016-08-31 20:00:25 CST. --
Sep 07 19:25:01 localhost.localdomain systemd[1]: Started Device-mapper event daemon.
Sep 07 19:25:01 localhost.localdomain systemd[1]: Starting Device-mapper event daemon...
Sep 07 19:25:01 localhost.localdomain dmeventd[25315]: dmeventd ready for processing.
Sep 07 19:25:01 localhost.localdomain lvm[25315]: Monitoring thin centos-thinpool.
```
查看主机上的devicemapper组织结构：
```bash
#lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda 8:0 0 835.4G 0 disk
├─sda1 8:1 0 200M 0 part /boot/efi
├─sda2 8:2 0 500M 0 part /boot
└─sda3 8:3 0 834.7G 0 part
├─centos-root 253:0 0 50G 0 lvm /
├─centos-swap 253:1 0 17.7G 0 lvm [SWAP]
├─centos-thinpool_tmeta 253:2 0 2G 0 lvm
 │ └─centos-thinpool 253:6 0 200G 0 lvm
 ├─centos-thinpool_tdata 253:4 0 200G 0 lvm
 │ └─centos-thinpool 253:6 0 200G 0 lvm
 └─centos-docker--volume 253:5 0 560G 0 lvm
```

### 5\.8 direct-lvm的数据卷扩容方法参见下面链接

[https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#/for-a-direct-lvm-mode-configuration](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#/for-a-direct-lvm-mode-configuration)

### 5\.9 Device Mapper与docker性能分析**

Device Mapper的两个重要特性allocate-on-demand and copy-on-write会对docker性能产生全面的影响。
*   docker默认使用的存储块大小为64KB，且是按docker容器使用需求动态分配。这对写入大量小文件的场景，会降低写性能。
*   写时复制功能，在修改image数据时会先复制一份到容器并修改，这对容器性能有显著影响。
*   写时复制功能，是基于64KB块操作的，如果要修改一个1GB文件中的一个64KB数据块，只会拷贝这一个64KB的数据块。
*   对于有大量小文件写（小于64KB）的使用场景，写时复制功能会明显降低容器性能。

其它影响docker容器性能的因素：

*   docker使用devicemapper驱动的模式，默认为loop-lvm，而生产环境建议更换为direct-lvm；
*   为获得更高容器性能，建议把Data file and Metadata file放置于SSD或其它高性能存储设备上；
*   devicemapper不是一种特别高效的docker存储驱动，它需要把容器的多层快照文件加载到内存中，这对于PaaS和高负载的场景来说都不适合；
*   关于data volumes数据磁盘的设置，这是容器最可靠的数据磁盘实现方式，它回避开了docker存储中的精简配置、写时复制等特性，如果应用会产生重负载的磁盘读写，则data volumes是最好的方式；

## 6\. docker容器的网络管理

断开一个容器的网络：
`$ docker network disconnect bridge networktest`

### 6\.1 查看docker的默认网络配置
```bash
# docker network ls
NETWORK ID NAME DRIVER SCOPE
b51472048f03 bridge bridge local
039b69c8fd33 host host local
8a5128a64ba6 none null local
```
*   bridge，除非使用以下参数选项指定其它网络，否则容器会默认使用默认的bridge网络。
*   none，代表一个没有配置网卡的容器网络环境。
*   host，会把容器的网络配置为和宿主机相同的网段。

`docker run --network=<NETWORK>`

从宿主机进入一个容器的方法：
`$ docker attach nonenetcontainer`

从容器退出，回到宿主机的方法：
`CTRL-p CTRL-q`

容器使用的默认网桥在宿主机系统中表现为docker0。

### 6\.2 启动两个测试容器以观察网络配置
```bash
docker run -itd --name=container1 busybox
docker run -itd --name=container2 busybox
```

查看默认网桥的详细信息：
`docker network inspect bridge`
进入每个容器观察其内部网络配置信息。

### 6\.3 创建自定义的桥接网络
```bash
docker network create --driver bridge user_net
aa1166a12df0b6eb3523391e04334962db70690a7f38f60c71b5a3f499149de4
```
使用以下命令观察网络配置的变化：
```bash
docker network inspect user_net
ip a
docker network ls
NETWORK ID NAME DRIVER SCOPE
b51472048f03 bridge bridge local
039b69c8fd33 host host local
8a5128a64ba6 none null local
aa1166a12df0 user_net bridge local
```
创建一个使用自定义网桥的容器：
```bash
docker run --network=user_net -itd --name=container3 busybox
3c9e75d09d486f73a4076e6e861a842942519108821afb17084c8cb3a8ed793d
```
手动把container1 容器加入到自定义网络中：
`$ docker network connect my-bridge-network container1`
可以看到container1 容器变成了双网卡

### 6\.4 创建一个docker集群模式下的overlay network

overlay network，覆盖网络就是应用层网络，它是面向应用层的，不考虑或很少考虑网络层，物理层的问题。
swarm mode，集群模式。
```bash
docker network create --driver overlay --subnet 10.0.9.0/24 my-multi-host-network
docker service create --replicas 2 --network my-multi-host-network --name my-web nginx
```
注：docker service创建的是运行在swarm mode模式下的容器，与使用docker run创建的容器互不连通。
注：replicas参数，决定同时启动几个容器实例。
[https://docs.docker.com/engine/swarm/](https://docs.docker.com/engine/swarm/)

### 6\.5 创建一个使用外部key-value服务的overlay network
可以使用Consul, Etcd, and ZooKeeper这些软件提供外部key-value服务。
docker集群模式的参考资料见下面链接。
[https://docs.docker.com/engine/userguide/networking/get-started-overlay/](https://docs.docker.com/engine/userguide/networking/get-started-overlay/)

### 6\.6 常用的docker网络管理命令
```bash
docker network create
docker network connect
docker network ls
docker network rm
docker network disconnect
docker network inspect
```

## 7\. 配置docker容器成为系统自启动服务
例如一个名为“tomcat_server”的容器，需要设置为随系统自启动的服务。
```bash
cd  /etc/systemd/system
vi  tomcat_server.service
[Unit]
Description=Tomcat container
Requires=docker.service
After=docker.service
 
[Service]
Restart=always
ExecStart=/usr/bin/docker start -a tomcat_server
ExecStop=/usr/bin/docker stop -t 2 tomcat_server
 
[Install]
WantedBy=default.target
 
systemctl daemon-reload
systemctl start tomcat_server.service
systemctl enable tomcat_server.service
```
注意：使用这种方式配置和使用systemctl启动的docker容器服务将不能使用docker stop关掉，从容器系统中执行exit时也不会关闭该容器。如果要关停该容器，只能使用systemctl stop tomcat_server.service 。

## 8\. 创建一个预编译安装了mysql5.6+jdk1.7+tomcat7的centos6.8系统dockerfile

### 8\.1 从官网获取一个centos6.8的镜像
`docker pull centos:6.8`

### 8\.2 上传制作dockerfile需要用到的各种软件
```bash
mkdir -p /opt/centos6-mysql-jdk-tomcat
 
cd /opt/centos6-mysql-jdk-tomcat
ls
apache-tomcat-7.0.70.tar.gz  Dockerfile   jdk-7u21-linux-x64.rpm  nload-0.7.4-1.el6.rf.x86_64.rpm  authorized_keys  iftop-1.0-0.7.pre4.el6.x86_64.rpm  mysql-5.6.26.tar.gz
```

### 8\.3 在Docker主机上创建本地密钥
```bash
cd   /opt/centos6-mysql-jdk-tomcat
touch authorized_keys
chmod 600 authorized_keys
cat /root/.ssh/id_rsa.pub > authorized_keys
```

### 8\.4 编写dockerfile文件
```bash
cd  /opt/centos6-mysql-jdk-tomcat
vi  Dockerfile
###########################################################
FROM centos:6.8
MAINTAINER watermelonbig <watermelonbig@163.com>
#Install users
RUN useradd -d /data -m operuser
RUN mkdir -p /data/backup
RUN mkdir -p /data/logs
RUN echo "operuser:Test2016" | chpasswd
RUN echo "root:Test2016" | chpasswd
RUN gpasswd -a operuser root
RUN chown -R operuser.operuser /data/
 
#Initialize system
RUN  yum -y install vim* wget gcc gcc-c++ automake autoconf vixie-cron ntp sysstat iotop system-config-firewall-tui system-config-lvm system-config-network-tui lsof tcpdump telnet traceroute mtr dos2unix
COPY ./nload-0.7.4-1.el6.rf.x86_64.rpm .
COPY ./iftop-1.0-0.7.pre4.el6.x86_64.rpm .
RUN  rpm -ivh nload-0.7.4-1.el6.rf.x86_64.rpm && rpm -ivh iftop-1.0-0.7.pre4.el6.x86_64.rpm
RUN  yum -y groupinstall "Chinese Support"
RUN  sed -i '/LANG/s/.*/LANG="zh_CN.UTF-8"/g' /etc/sysconfig/i18n
RUN  export LANG=zh_CN.UTF-8
RUN  echo 'LANG=zh_CN.UTF-8' > /etc/environment && echo 'LC_ALL=' >> /etc/environment
RUN  source /etc/environment
#RUN  localedef -v -c -i zh_CN -f UTF-8 zh_CN.UTF-8 > /dev/null 2>&1
 
RUN  service crond start && chkconfig crond on
RUN  echo '20 1 * * * /usr/sbin/ntpdate 2.cn.pool.ntp.org;/sbin/hwclock -w > /dev/null 2>&1' >> /tmp/crontab2.tmp
RUN  crontab /tmp/crontab2.tmp
RUN  rm -f /tmp/crontab2.tmp
 
RUN echo 'net.ipv6.conf.all.disable_ipv6 = 1' >> /etc/sysctl.conf && echo 'net.ipv6.conf.default.disable_ipv6 = 1' >> /etc/sysctl.conf && echo 'net.ipv4.ip_local_port_range = 1024 65000' >> /etc/sysctl.conf && echo 'vm.swappiness = 10' >> /etc/sysctl.conf
 
#change ulimit
RUN  echo '* soft nofile 65535' >> /etc/security/limits.conf && echo '* hard nofile 65535' >> /etc/security/limits.conf
RUN  echo "session   required        /lib64/security/pam_limits.so" >> /etc/pam.d/login
RUN  sed -i 's/1024/65535/g' /etc/security/limits.d/90-nproc.conf
 
#Monitor
RUN  yum -y install net-snmp net-snmp-libs net-snmp-utils net-snmp-devel
RUN  echo 'com2sec    company     default    companymonitor' > /etc/snmp/snmpd.conf && echo 'group      companyg    v2c        company' >> /etc/snmp/snmpd.conf && echo 'view       all      included   .1 80' >> /etc/snmp/snmpd.conf && echo 'access companyg "" any noauth exact all none none' >> /etc/snmp/snmpd.conf
RUN  service snmpd start && chkconfig snmpd on
 
#Install openssh
RUN yum install -y openssh-server
RUN mkdir /root/.ssh
COPY ./authorized_keys /root/.ssh/authorized_keys
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config
RUN sed -i '/UseDNS yes/s/.*/UseDNS no/g' /etc/ssh/sshd_config
RUN chkconfig sshd on && service sshd start
RUN chmod u+s /usr/sbin/lsof && chmod u+s /usr/sbin/tcpdump && chmod u+s /usr/sbin/mtr
 
#Install MySQL
RUN yum install -y wget make gcc gcc-c++ cmake bison-devel ncurses ncurses-devel openssl openssl-devel libtool*
RUN useradd -d /data/mysql -m mysql
RUN mkdir -p /usr/local/mysql
COPY ./mysql-5.6.26.tar.gz .
RUN tar zxf mysql-5.6.26.tar.gz
RUN cd mysql-5.6.26 && cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITHOUT_FEDERATED_STORAGE_ENGINE=1 -DENABLE_DOWNLOADS=1 -DENABLED_LOCAL_INFILE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock -DMYSQL_TCP_PORT=3306 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_SSL=yes -DWITH_DEBUG=0 && make && make install
 
RUN chown -R mysql:mysql /usr/local/mysql
RUN chmod 755 /data
RUN cd /usr/local/mysql/scripts && chmod 755 mysql_install_db
RUN /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/mysql --user=mysql
RUN cd /usr/local/mysql && cp ./my.cnf /etc/my.cnf && cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
RUN echo 'explicit_defaults_for_timestamp=true' >> /etc/my.cnf
RUN chmod 755 /etc/init.d/mysqld && chkconfig mysqld on
 
#Install JDK
RUN yum -y install iputils
COPY ./jdk-7u21-linux-x64.rpm .
RUN rpm -Uvh jdk-7u21-linux-x64.rpm
RUN echo 'export JAVA_HOME=/usr/java/jdk1.7.0_21' >> /etc/profile && echo 'export PATH=/usr/local/mysql/bin:$JAVA_HOME/bin:$PATH' >> /etc/profile &&  echo 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar' >> /etc/profile
RUN source /etc/profile
 
#Install Tomcat
COPY ./apache-tomcat-7.0.70.tar.gz .
RUN tar zxf apache-tomcat-7.0.70.tar.gz -C /data/
RUN cd /data/ && mv apache-tomcat-7.0.70 tomcat-7.0.70
RUN chown -R operuser.operuser /data/tomcat-7.0.70
RUN cd / && rm -rf jdk-7u21-linux-x64.rpm mysql-5.6.26*  apache-tomcat-7.0.70* iftop-1.0-0.7.pre4.el6.x86_64.rpm nload-0.7.4-1.el6.rf.x86_64.rpm
 
#start sshd service and initialize system virables
CMD /etc/init.d/sshd start;source /etc/profile;/bin/bash
 
EXPOSE 8080 3306 22
```

### 8\.5 执行创建镜像命令
```bash
cd  /opt/centos6-mysql-jdk-tomcat
docker build -t centos6.8-and-mysql5.6  .
```
看到以下信息时，即镜像创建成功：
*Successfully built b527f89dbc90*

```bash
# docker images
REPOSITORY               TAG                 IMAGE ID            CREATED              SIZE
centos6.8-and-mysql5.6   latest              b527f89dbc90        About a minute ago   5.13 GB
centos                   6.8                 0cd976dc0a98        9 days ago           194.5 MB
```

**新建一个容器测试下**：
交互式、后台运行，进行端口转发，挂载了一个数据卷到/data
```bash
# docker run -itd -p 8080:8080 -p 3306:3306 -p 10022:22 -v /data --name newserver centos6-mysql5.6-jdk7-tomcat7:latest
981fdd240b3f3c5015542c9a8ae7716700f27723fd3f912fd2b3848105c5242b
# docker ps
CONTAINER ID        IMAGE                                  COMMAND             CREATED             STATUS              PORTS                                                                   NAMES
981fdd240b3f        centos6-mysql5.6-jdk7-tomcat7:latest   "/bin/bash"         8 seconds ago       Up 7 seconds        0.0.0.0:3306->3306/tcp, 0.0.0.0:8080->8080/tcp, 0.0.0.0:10022->22/tcp   newserver
```

进入到容器系统中：
`# docker attach newserver`
注意：上面这种方式进入容器，在退出时如果直接执行exit将退出容器的同时，也把容器关闭。如果不希望关闭容器，执行ctrl+p ctrl+q退出。或者通过外部SSH远程登录容器系统。

**关于系统字符集编码的设置**：
在创建镜像的过程中已经设置为使用zh_CN.UTF-8字符集，但进入容器后切换用户时会遇到一个报错，像下面这样。
```bash
[root@981fdd240b3f /]# su - operuser
-bash: warning: setlocale: LC_CTYPE: cannot change locale (zh_CN.UTF-8): No such file or directory
-bash: warning: setlocale: LC_COLLATE: cannot change locale (zh_CN.UTF-8): No such file or directory
-bash: warning: setlocale: LC_MESSAGES: cannot change locale (zh_CN.UTF-8): No such file or directory
-bash: warning: setlocale: LC_NUMERIC: cannot change locale (zh_CN.UTF-8): No such file or directory
-bash: warning: setlocale: LC_TIME: cannot change locale (zh_CN.UTF-8): No such file or directory
```

没有找到特别好的解决办法，未能通过镜像文件本身解决。目前的解决方法是进入容器后，执行下面命令：
`localedef -v -c -i zh_CN -f UTF-8 zh_CN.UTF-8`

**关于mysql的初始化设置**：
mysql安装后的初始root用户密码为空，建议在创建新容器后执行以下命令重置mysql密码以及进行安全加固。
`/usr/local/mysql/bin/mysql_secure_installation`

**观察容器的存储、网络和启动的服务以更深入理解docker：**
上面我们已经启动了一个容器实例newserver，下面我们再创建一个容器的实例testserver。
这两个容器均是基于相同的镜像创建的，可以进入容器把它们内部的mysql、tomcat服务都启动。再从外部通过转发端口访问这两个容器实例和它们的应用服务。
`docker run -itd -p 8081:8080 -p 3307:3306 -p 10023:22 -v /data --name testserver centos6-mysql5.6-jdk7-tomcat7:latest`
在docker宿主机系统中，观察docker网络、进程方面的信息变化。

## 9\. docker容器的cpu和内存资源使用限制
下面的参数可以用来调整container内的性能参数。
```bash
-m="": Memory limit (format: <number><optional unit>, where unit = b, k, m or g)
-c=0 : CPU shares (relative weight)
```
通过docker run -m 可以很方便的调整container所使用的内存资源。如果host支持swap内存，那么使用-m可以设定比host物理内存还大的值。
同样，通过-c 可以调整container的cpu优先级。默认情况下，所有的container享有相同的cpu优先级和cpu调度周期。但你可以通过Docker来通知内核给予某个或某几个container更多的cpu计算周期。
默认情况下，使用-c或者--cpu-shares 参数值为0，可以赋予当前活动container 1024个cpu共享周期。这个0值可以针对活动的container进行修改来调整不同的cpu循环周期。

比如，我们使用-c或者--cpu-shares =0启动了C0，C1，C2三个container，使用-c/--cpu-shares=512启动了C3.这时，C0，C1，C2可以100%的使用 CPU资源(1024)，但C3只能使用50%的CPU资源(512)。如果这个host的OS是时序调度类型的，每个CPU时间片是100微秒，那么 C0，C1，C2将完全使用掉这100微秒，而C3只能使用50微秒。

我们把前面几步中创建的docker容器newserver删除掉（docker rm），记得把创建的存储卷也删了（docker volume rm），然后使用以下的命令重新创建这个容器：
`docker run -itd -m=12g -c=512 -p 8080:8080 -p 3306:3306 -p 10022:22 -v /data --name newserver centos6-mysql5.6-jdk7-tomcat7:latest`
注：上面的容器，cpu资源最大不能超过总资源的一半；内存最大可以使用12GB。

## 10\. docker镜像的导出与导入管理

导出镜像：
```bash
# docker save centos6-mysql5.6-jdk7-tomcat7 > /opt/centos6-mysql5.6-jdk7-tomcat7.tar
# ls -lh /opt
-rw-r--r--. 1 root  root  4.9G Sep  9 09:39 centos6-mysql5.6-jdk7-tomcat7.tar
```

可以拷贝到其它docker主机上导入后使用：
`# docker load < /opt/centos6-mysql5.6-jdk7-tomcat7.tar`

## 11\. 使用docker commit命令提交一个新的镜像
commit主要用于通过差异性提交，创建一个新的image。
Usage: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]Create a new image from a container's changes
*   -a, --author="" Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
*   -m, --message="" Commit message
*   -p, --pause=true Pause container during commit

默认情况下，在提交时，容器的执行将被暂停，以保证数据的完整性，当然你可以指定选项 p ，来禁止。
例如：
`docker commit 1ebbcbe08ea5 watermelonbig/jenkins:version1`

## 12\. 为已经创建的容器新增映射端口
一个容器IP是172.17.0.2，下面为其增加一个转发端口：18081--->8081
```bash
iptables -t nat -A DOCKER -p tcp -m tcp --dport 18081 -j DNAT --to-destination 172.17.0.2:8081
iptables -t nat -A POSTROUTING -p tcp -m tcp -s 172.17.0.2 -d 172.17.0.2 --dport 8081 -j MASQUERADE
iptables -A DOCKER -p tcp -m tcp -d 172.17.0.2 --dport 8081 -j ACCEPT
```