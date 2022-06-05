## 注意事项

### 可上网机器

1、安装wget

```shell
yum -y install wget
```

2、安装yum-utils

```shell
yum -y install yum-utils
```



## 一、简述

离线在 Centos 中部署 DockerCE

## 二、操作步骤

### 可上网机器 A（最好是纯净的系统）：

 1、配置安装源存放路径

```shell
mkdir -p /root/docker-ce-local && cd /root/docker-ce-local
```

2、获取 createrepo 安装包

```shell
yum install --downloadonly --downloaddir=/root/docker-ce-local createrepo
```

3、获取系统更新 yum 源

```shell
yum update --downloadonly --downloaddir=/root/docker-ce-local
```

*4、卸载旧版本（如果没有安装过 docker 跳过）

```shell
yum remove docker docker-common docker-selinux docker-engine
```

5、获取 docker-ce 所需依赖

```shell
yum install --downloadonly --downloaddir=/root/docker-ce-local yum-utils device-mapper-persistent-data lvm2
```

6、设置 docker-ce 在线存储库

```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

7、更新 yum 源索引

```shell
yum makecache fast
```

​	centos8请使用`yum makecache`

8、获取 docker-ce 及相关 rpm 安装源

```shell
yum install --downloadonly --downloaddir=/root/docker-ce-local docker-ce
```

9、查看安装时 docker 所需要的密钥并下载

```shell
more /etc/yum.repos.d/docker-ce.repo
cd /root/docker-ce-local/
wget https://download.docker.com/linux/centos/gpg
```

10、安装 createrepo

```shell
yum install createrepo
```

11、初始化源文件的 repodata

```shell
createrepo -pdo /root/docker-ce-local /root/docker-ce-local
createrepo --update /root/docker-ce-local
```

12、将文件夹打包为 yum-local.tgz

```shell
cd /root
tar -zcvf centos-local.tgz docker-ce-local/
```



### 不可上网机器 B：

1、在目标计算机上将 tgz 包上传至/root 路径下，并解压 centos-local.tgz 文件

```shell
cd /root
tar -xvzf centos-local.tgz
```

2、安装 createrepo

```shell
cd /root/docker-ce-local
rpm -ivh createrepo-0.9.9-28.el7.noarch.rpm
```

注：createrepo 版本可能不一样，根据自己下载的包的版本安装

3、备份安装源，将所有的安装源移动到备份文件夹中

```shell
cd /etc/yum.repos.d/
mkdir repobak
mv CentOS* repobak/
```

4、新增 docker-ce-local.repo 源文件，写入以下内容

```shell
vi /etc/yum.repos.d/docker-ce-local.repo
[docker-ce-local]
name=Local Yum
baseurl=file:///root/docker-ce-local/
gpgcheck=1
gpgkey=file:///root/docker-ce-local/gpg
enabled=1
```

5、生成源索引及缓存

```shell
createrepo /root/docker-ce-local
yum makecache
```

6、安装 docker-ce

```shell
yum install docker-ce
```

7、启动并测试

```shell
systemctl start docker.service
docker version
```

需要注意的是机器 A 和 B 一定要系统版本一致，否则会出现缺少依赖

也可以缺少什么找对应的 rpm 包也是可以的