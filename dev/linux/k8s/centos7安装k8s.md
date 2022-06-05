# kubeadm安装k8s 1.22.2版本



## 一、本次操作系统说明

### 操作系统说明

```shell
[root@master-101 ~]# cat /proc/version
Linux version 3.10.0-957.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC) ) #1 SMP Thu Nov 8 23:39:32 UTC 2018
[root@master-101 ~]# rpm -q centos-release
centos-release-7-6.1810.2.el7.centos.x86_64
[root@master-101 ~]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

### docker 版本说明

```shell
[root@master-101 ~]# docker --version
Docker version 20.10.9, build c2ea9bc
```

### kubernetes版本

```shell
[root@master-101 ~]# kubelet --version
Kubernetes v1.22.2
```

### 服务器准备说明

```
192.168.154.101 master-101
192.168.154.111 node-111
```

### <font color='red'>操作说明</font>

​	<font color='red'>**二、三所有master、node节点都需要执行**</font>

## 二、配置系统

```shell
#停止、关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#关闭 SeLinux
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
setenforce 0

#关闭swap
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

#设置时区
timedatectl set-timezone Asia/Shanghai
yum install ntpdate -y
#同步时间
ntpdate pool.ntp.org

#桥接流量
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

# 设置主机名
hostnamectl set-hostname xx

#在两台机器上打开/etc/hosts加入
vi /etc/hosts
192.168.154.101 master-101
192.168.154.111 node-111
```

## 三、系统准备

### 安装 wget

```shell
yum install wget
```

### 安装docker

```shell
wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce

# 设置Docker镜像加速器
mkdir -p /etc/docker/
cat > /etc/docker/daemon.json <<EOF
{
"exec-opts":["native.cgroupdriver=systemd"],
"registry-mirrors": ["http://hub-mirror.c.163.com"]
}
EOF

systemctl start docker & systemctl enable docker
```

### 安装k8s

```shell
cat > /etc/yum.repos.d/kubernetes.repo <<EOF 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl
systemctl enable kubelet
```



## 四、初始化集群

192.168.177.6是本机ip，其他地址是k8规划地址。k8版本，按照安装的版本来填

```shell
kubeadm init \
--apiserver-advertise-address=192.168.154.101 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.2 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=10.88.0.0/16

#初始化成功后，记住 kubeadm join xxx 这句话，node节点加入需要


docker pull coredns/coredns
# 镜像拉不下来，错误信息如下，因为阿里云这没有此镜像，需要手动下载镜像，然后打tag
# [ERROR ImagePull]: failed to pull image registry.aliyuncs.com/google_containers/coredns:v1.8.4: output: Error response from daemon: manifest for registry.aliyuncs.com/google_containers/coredns:v1.8.4 not found: manifest unknown: manifest unknown
https://blog.51cto.com/8999a/2784605
#错误则下面语句执行
docker tag coredns/coredns:latest registry.aliyuncs.com/google_containers/coredns:v1.8.4

# 查看初始化集群时，需要拉的镜像名
kubeadm config images list
```

### kubernetes 初始化错误解决方法

```shell
# master 初始化错误解决方法
[root@master ~]# kubeadm init --kubernetes-version=v1.20.0 --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=10.0.0.7
[init] Using Kubernetes version: v1.20.0
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.0. Latest validated version: 19.03
        [WARNING Hostname]: hostname "master" could not be reached
        [WARNING Hostname]: hostname "master": lookup master on 223.6.6.6:53: no such host
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher

# 解决方法

错误一：
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at 
解决方法： "exec-opts": ["native.cgroupdriver=systemd"]
加入下面一行 
[root@master ~]# vim /etc/docker/daemon.json
  "registry-mirrors": ["https://q2hy3fzi.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

错误二：
[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.0. Latest validated version: 19.03
解决方法：
这个错误可以忽略，是因为docker版本太新造成的

错误三：
[WARNING Hostname]: hostname "master" could not be reached
解决方法：
所有节点修改host文件
[root@master ~]# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.7 master
10.0.0.17 node1
10.0.0.27 node2

错误四：
[ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
解决方法：
关闭swap 
修改fstab文件
[root@master ~]# swapoff -a
[root@master ~]# vim /etc/fstab
# /etc/fstab
# Created by anaconda on Tue Jul 21 11:31:26 2020
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
UUID=92d0df55-7937-4cef-9df7-daf5cd77f916 /                       xfs     defaults        0 0
UUID=6f2be111-89b2-4f3a-a015-fe3e6c5290fc /boot                   xfs     defaults        0 0
UUID=47913215-9063-4ff4-a56b-0df738def1bc /data                   xfs     defaults        0 0
#UUID=7ca8f821-027c-44d6-8eac-a4a83b409087 swap                    swap    defaults        0 0
————————————————
版权声明：本文为CSDN博主「Peak traversing」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_45054797/article/details/112159446

错误五：
[WARNING Hostname]: hostname "master-101": lookup master-101 on 192.168.154.2:53: server misbehaving
#修改虚拟机中/etc/resolv.conf
nameserver 8.8.8.8

错误六：
 failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
 请确认docker是否设置的镜像加速
```

### 失败后重新初始化

```shell
kubeadm reset
```



### 安装flannel网络

目的是为了集群之间pod通信，也有其他方案，这里选择flannel

```shell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

```shell
kubectl apply -f kube-flannel.yml
```

### 安装dashboard

```shell
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```

```shell
kubectl apply -f recommended.yml
```



## 五、节点加入集群

1、执行`master`节点init的时候输出的`kubeadm join`,即可

```shell
kubeadm join 192.168.154.101:6443 --token tp3h98.bgrewdo35ajuciz1 \
        --discovery-token-ca-cert-hash sha256:64b83705bc9c9dd0ad4ebaab0ab6e829474de46c4e9608919a89133a7edf296e
```

​	如果在部署`master`节点的时候没有保存, 则可以通过`kubeadm token list`找回, `ip`即为`master`节点所在机器的ip, 端口为6443(可能默认是)

2、生成dashboard登陆的token

```shell
kubectl create serviceaccount dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
kubectl get secret -n kube-system
# 找到dashboard-admin-token-klk6z
kubectl describe secret dashboard-admin-token-klk6z -n kube-system
# 把token输入，登陆
```

## 六、校验节点状态

### 查看节点状态

```shell
kubectl get nodes
```

所有的节点皆为`Ready`状态表示集群正常

```shell
[root@master-101 k8s]# kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
master-101   Ready    control-plane,master   42m   v1.22.2
node-111     Ready    <none>                 23m   v1.22.2
```



### 查看pod状态

```shell
kubectl get pods --all-namespaces -o wide
```

所有的的组件的`READY`皆为1/1

```
[root@master-101 k8s]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE   IP                NODE         NOMINATED NODE   READINESS GATES
kube-system   coredns-7f6cbbb7b8-dh526             0/1     Running   0          43m   10.88.1.2         node-111     <none>           <none>
kube-system   coredns-7f6cbbb7b8-m67wh             1/1     Running   0          43m   10.88.0.2         master-101   <none>           <none>
kube-system   etcd-master-101                      1/1     Running   1          43m   192.168.154.101   master-101   <none>           <none>
kube-system   kube-apiserver-master-101            1/1     Running   1          43m   192.168.154.101   master-101   <none>           <none>
kube-system   kube-controller-manager-master-101   1/1     Running   1          43m   192.168.154.101   master-101   <none>           <none>
kube-system   kube-flannel-ds-gjm5k                1/1     Running   0          18m   192.168.154.111   node-111     <none>           <none>
kube-system   kube-flannel-ds-rdc78                1/1     Running   0          18m   192.168.154.101   master-101   <none>           <none>
kube-system   kube-proxy-sd2z9                     1/1     Running   0          43m   192.168.154.101   master-101   <none>           <none>
kube-system   kube-proxy-vjr4t                     1/1     Running   0          25m   192.168.154.111   node-111     <none>           <none>
kube-system   kube-scheduler-master-101            1/1     Running   1          43m   192.168.154.101   master-101   <none>           <none>
```

coredns-7f6cbbb7b8-dh526 状态为0 是因为子节点kubectl命令需要使用kubernetes-admin来运行



### 错误解决方案

​	1、子节点 不能使用 kubectl 命令，例：

```shell
[root@node-111 ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

​	解决：

​		(1)、将主节点中的【/etc/kubernetes/admin.conf】文件拷贝到从节点相同目录下：

​		(2)、配置环境变量

```shell
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
```

​		(3)、立即生效

```shell
source ~/.bash_profile
```

​	在运行kubectl命令就成功，同时查看pod状态，所有状态都已经是``Ready``了

## 七、删除节点

### 在master节点执行

```shell
kubectl drain node-111  --delete-local-data --force --ignore-daemonsets
kubectl delete node node-111
```

​	node-111:节点名称

### 在移除的节点上执行

```
kubeadm reset
```

​     