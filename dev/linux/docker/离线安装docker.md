# 下载rpm镜像：

​	1、rpm镜像下载地址用的阿里的 ：https://mirrors.aliyun.com/docker-ce/linux/centos/

​	2、官方下载：http://docker-release-red-prod.s3-website-us-east-1.amazonaws.com/linux/centos/7/x86_64/stable/Packages/

根据centos版本下载以下三个包：

```
containerd.io-1.2.13-3.2.el7.x86_64.rpm
docker-ce-19.03.13-3.el7.x86_64.rpm
docker-ce-cli-19.03.13-3.el7.x86_64.rpm
```

下载container-selinux

http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.107-3.el7.noarch.rpm

下载policycoreutils-python

http://www.rpmfind.net/linux/opensuse/distribution/leap/15.3/repo/oss/x86_64/policycoreutils-python-2.6-5.3.8.x86_64.rpm

## policycoreutils-python需要的依赖：

下载 audit-libs-python-2.8.5-4.el7.x86_64.rpm

https://vault.centos.org/7.6.1810/cr/x86_64/Packages/audit-libs-python-2.8.5-4.el7.x86_64.rpm

下载policycoreutils

http://www.rpmfind.net/linux/fedora/linux/development/rawhide/Everything/x86_64/os/Packages/p/policycoreutils-3.2-2.fc35.x86_64.rpm

下载python-enum34

https://vault.centos.org/7.6.1810/os/Source/SPackages/python-enum34-1.0.4-1.el7.src.rpm

下载：python-ipy

http://mirror.centos.org/centos-7/7/os/x86_64/Packages/python-IPy-0.75-6.el7.noarch.rpm

下载：python-selinux

https://sourceforge.net/projects/python-selinux/files/python-selinux/

下载：checkpolicy

http://rpmfind.net/linux/rpm2html/search.php?query=checkpolicy

下载：libcgroup

http://www.rpmfind.net/linux/rpm2html/search.php?query=libcgroup(x86-64)

下载：libsemanage-python

http://www.rpmfind.net/linux/rpm2html/search.php?query=libsemanage-python&submit=Search+...&system=&arch=

下载：setools-libs

http://www.rpmfind.net/linux/rpm2html/search.php?query=setools-libs



libc.so.6

# 2、安装

上传到服务器，将这三个包按照顺序分别安装即可，顺序的话，安装的时候会提示，如果顺序错了会提示有依赖包

```
rpm -ivh policycoreutils-python-2.6-5.3.8.x86_64.rpm

rpm -ivh container-selinux-2.107-3.el7.noarch.rpm

rpm -ivh containerd.io-1.2.13-3.2.el7.x86_64.rpm 

rpm -ivh docker-ce-cli-19.03.13-3.el7.x86_64.rpm

rpm -ivh docker-ce-19.03.13-3.el7.x86_64.rpm 
```



# 3、测试

完成安装后，开启服务，查看是否安装成功

## 启动docker

> systemctl start docker

## 设置docker开机启动

> systemctl  enable docker.service

## 查看docker版本

> docker -v

