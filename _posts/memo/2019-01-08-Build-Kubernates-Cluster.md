---
layout: post
title: "Kubernates集群环境搭建"
date:       2019-01-08 10:00:00 +0800
author: "Jay Cai"
header-style: text
catalog: true
tags:
  - Kubernates
  - Docker
  - memo
---

## setup machines
### set hostname
```bash
[root@k8s-master ~]# hostnamectl set-hostname k8s-master
[root@k8s-master ~]# hostname
k8s-master
[root@k8s-master ~]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
cp: ‘/usr/share/zoneinfo/Asia/Shanghai’ and ‘/etc/localtime’ are the same file
[root@k8s-master ~]#
```

### change the ip addr to static
```bash
[root@k8s-master ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33`
+ BOOTPROTO="static"
+ IPADDR=172.168.10.110
+ GATEWAY=172.168.10.2
+ NETMASK=255.255.255.0
+ DNS1=172.168.10.2
[root@k8s-master ~]# systemctl restart network
```

### install vmware tools
```bash
[root@k8s-master ~]# yum -y install perl gcc net-tools kernel-devel
[root@k8s-master ~]#  reboot
[root@k8s-master ~]# uname -r
3.10.0-957.1.3.el7.x86_64
[root@k8s-master ~]# cd /lib/modules/3.10.0-957.1.3.el7.x86_64/build/include/
[root@k8s-master include]# ls
generated    linux
[root@k8s-master include]# cp generated/uapi/linux/version.h  linux/
[root@k8s-master include]# cd ~
[root@k8s-master ~]# mkdir -p /mnt/cdrom
[root@k8s-master ~]# mount /dev/cdrom /mnt/cdrom
mount: /dev/sr0 is write-protected, mounting read-only
[root@k8s-master ~]# ls /mnt/cdrom/
manifest.txt     VMwareTools-10.1.6-5214329.tar.gz  vmware-tools-upgrader-64
run_upgrader.sh  vmware-tools-upgrader-32
[root@k8s-master ~]# cp /mnt/cdrom/VMwareTools-10.1.6-5214329.tar.gz /root/
[root@k8s-master ~]# ls
anaconda-ks.cfg  VMwareTools-10.1.6-5214329.tar.gz
[root@k8s-master ~]# umount /mnt/cdrom
[root@k8s-master ~]# tar -zxvf VMwareTools-10.1.6-5214329.tar.gz
[root@k8s-master ~]# cd vmware-tools-distrib
[root@k8s-master vmware-tools-distrib]# ./vmware-install.pl
[root@k8s-master vmware-tools-distrib]# reboot
```

### install telnet ftp
```bash
[root@k8s-master ~]# yum -y install xinetd telnet telnet-server vsftpd
[root@k8s-master ~]# systemctl enable xinetd.service
[root@k8s-master ~]# systemctl enable telnet.socket
Created symlink from /etc/systemd/system/sockets.target.wants/telnet.socket to /usr/lib/systemd/system/telnet.socket.
[root@k8s-master ~]# systemctl enable vsftpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service to /usr/lib/systemd/system/vsftpd.service.
[root@k8s-master ~]# systemctl start telnet.socket
[root@k8s-master ~]# systemctl start xinetd
[root@k8s-master ~]# systemctl start vsftpd.service
```

### install jdk&scala
```bash
[root@k8s-master ~]# wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "https://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.rpm"
[root@k8s-master ~]# rpm -ivh jdk-8u191-linux-x64.rpm
[root@k8s-master ~]# rpm -ivh scala-2.12.8.rpm
```

#### find the install path
```bash
[root@k8s-master ~]# rpm -qpl jdk-8u191-linux-x64.rpm
[root@k8s-master ~]# vim /etc/profile
```
+ JAVA_HOME=/usr/java/jdk1.8.0_191-amd64
+ SCALA_HOME=/usr/share/scala
+ PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin
+ export JAVA_HOME SCALA_HOME PATH
```bash
[root@k8s-master ~]# source /etc/profile
```

### create user
```bash
[root@k8s-master ~]# groupadd hadoop
[root@k8s-master ~]# adduser -g hadoop hadoop
[root@k8s-master ~]# passwd hadoop
Changing password for user hadoop.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[root@k8s-master ~]# groups hadoop
hadoop : hadoop
[root@k8s-master ~]# chage -l hadoop
Last password change                                    : Jan 08, 2019
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[root@k8s-master ~]# vim /etc/sudoers
[root@k8s-master ~]#
```

## Install Docker CE
### Uninstall old versions
```bash
[root@k8s-master ~]# yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
    docker-engine
```

### Set up the repository
#### Install required packages.
```bash
[root@k8s-master ~]# yum install yum-utils device-mapper-persistent-data lvm2
```

#### Add docker repository.
```bash
[root@k8s-master ~]# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### Install docker ce.
```bash
[root@k8s-master ~]# yum install docker-ce-18.06.1.ce
```

### config docker
#### confirm the cgroup driver
```bash
[root@k8s-master ~]# docker info
Cgroup Driver: cgroupfs
```

#### Create /etc/docker directory.
```bash
[root@k8s-master ~]# mkdir /etc/docker
```

### Setup docker daemon.
```bash
[root@k8s-master ~]# cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["http://f1361db2.m.daocloud.io"],
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

```bash
[root@k8s-master ~]# mkdir -p /etc/systemd/system/docker.service.d
```

### Restart docker.
```bash
[root@k8s-master ~]# systemctl daemon-reload
[root@k8s-master ~]# systemctl restart docker
[root@k8s-master ~]# systemctl enable docker
```
## Installing kubeadm, kubelet and kubectl
### set kubernets repos
```bash
[root@k8s-master ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```

### Set SELinux in permissive mode (effectively disabling it)
```bash
[root@k8s-master ~]# setenforce 0
[root@k8s-master ~]# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### disable the firewall
```bash
[root@k8s-master ~]# systemctl stop firewalld
[root@k8s-master ~]# systemctl disable firewalld
```

### swapoff
```bash
[root@k8s-master ~]# swapoff -a
[root@k8s-master ~]# sed -i 's/\(.*swap.*\)/# \1/g' /etc/fstab
```
### install
```bash
[root@k8s-master ~]# yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### start kubelet service
```bash
[root@k8s-master ~]# systemctl enable kubelet && systemctl start kubelet
```

### set sysctl config
```bash
[root@k8s-master ~]# cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Configure cgroup driver used by kubelet on Master Node(default:cgroupfs)
make sure the cgroup driver is same to docker
```bash
[root@k8s-master ~]# vim /etc/default/kubelet
```
+ KUBELET_EXTRA_ARGS=--cgroup-driver=<value>
```bash
[root@k8s-master ~]# systemctl daemon-reload
[root@k8s-master ~]# systemctl restart kubelet
```


## Clone the VM
### update the cloned vms
+ hostname
+ ip addr

### set password-less login
#### add ip addr to hosts every node
```bash
[root@k8s-master ~]# vim /etc/hosts
```
+ 172.168.10.110    k8s-master
+ 172.168.10.111    k8s-worker01
+ 172.168.10.112    k8s-worker02

#### set ssh on every node
```bash
ssh-keygen -t rsa -P ''
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
scp ~/.ssh/id_rsa.pub hadoop@k8s-worker01:~/.ssh/id_rsa.pub.master
scp ~/.ssh/id_rsa.pub hadoop@k8s-worker02:~/.ssh/id_rsa.pub.master
```

```bash
cat ~/.ssh/id_rsa.pub.master >> ~/.ssh/authorized_keys
```

```bash
chmod 600 ~/.ssh/authorized_keys
```


## create kubernetes cluster
### Initializing master
```bash
kubeadm init --kubernetes-version v1.13.1 --pod-network-cidr=192.168.0.0/16   --apiserver-advertise-address=172.168.10.110
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### install a pod network
```bash
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

### Control plane node isolation
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Joining your nodes
```bash
kubeadm join 172.168.10.110:6443 --token w5zf6q.drt7qra1fav782m1 --discovery-token-ca-cert-hash sha256:612f407f159fa1a2e4efe25a79f06ea784708acc8039dccba79bd3f421655185
```


## test the cluster
### create a pod with busybox for test
```bash
[hadoop@k8s-master ~]$ kubectl create -f https://k8s.io/examples/admin/dns/busybox.yaml
pod/busybox created
[hadoop@k8s-master ~]$ kubectl create deployment nginx --image=nginx:alpine
deployment.apps/nginx created
[hadoop@k8s-master ~]$ kubectl scale deployment nginx --replicas=2
deployment.extensions/nginx scaled
[hadoop@k8s-master ~]$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
busybox                  1/1     Running   1          33m     192.168.1.3   k8s-worker01   <none>           <none>
nginx-54458cd494-q8bkg   1/1     Running   0          3m44s   192.168.1.4   k8s-worker01   <none>           <none>
nginx-54458cd494-z2dj2   1/1     Running   0          3m54s   192.168.2.2   k8s-worker02   <none>           <none>
[hadoop@k8s-master ~]$ kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed
[hadoop@k8s-master ~]$ kubectl get services nginx
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.103.239.109   <none>        80:30109/TCP   18s
[hadoop@k8s-master ~]$ curl http://172.168.10.112:30109
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[hadoop@k8s-master ~]$ curl http://172.168.10.111:30109
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[hadoop@k8s-master ~]$ kubectl exec -ti busybox -- nslookup kubernetes.default
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
[hadoop@k8s-master ~]$ kubectl exec -ti busybox -- nslookup www.github.com
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.github.com
Address 1: 192.30.253.113 lb-192-30-253-113-iad.github.com
Address 2: 192.30.253.112 lb-192-30-253-112-iad.github.com
[hadoop@k8s-master ~]$ kubectl exec -ti busybox -- nslookup www.google.com
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.google.com
Address 1: 2404:6800:4007:80f::2004 maa05s05-in-x04.1e100.net
Address 2: 172.217.26.196 maa03s23-in-f196.1e100.net
[hadoop@k8s-master ~]$ kubectl exec -ti busybox -- nslookup www.baidu.com
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.baidu.com
Address 1: 104.193.88.123
Address 2: 104.193.88.77
[hadoop@k8s-master ~]$ kubectl exec -ti busybox -- nslookup nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx
Address 1: 10.103.239.109 nginx.default.svc.cluster.local
[hadoop@k8s-master ~]$ kubectl run curl --rm -it --image=radial/busyboxplus:curl
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
[ root@curl-66959f6557-qmnjt:/ ]$ nslookup www.github.com
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.github.com
Address 1: 192.30.253.112 lb-192-30-253-112-iad.github.com
Address 2: 192.30.253.113 lb-192-30-253-113-iad.github.com
[ root@curl-66959f6557-qmnjt:/ ]$ nslookup nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx
Address 1: 10.103.239.109 nginx.default.svc.cluster.local
[ root@curl-66959f6557-qmnjt:/ ]$ curl http://nginx/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ root@curl-66959f6557-qmnjt:/ ]$ curl http://192.168.1.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ root@curl-66959f6557-qmnjt:/ ]$ curl http://192.168.2.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ root@curl-66959f6557-qmnjt:/ ]$ exit
Session ended, resume using 'kubectl attach curl-66959f6557-r67ts -c curl -i -t' command when the pod is running
deployment.apps "curl" deleted
[hadoop@k8s-master ~]$
```
