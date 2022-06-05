### 一、下载centos镜像

下载自己需要的版本TAG，详见：

**[docker安装指定版本TAG的镜像](https://www.cnblogs.com/ztone/articles/10557486.html)**

```shell
$ sudo docker pull centos:centos7
```

### 二、下载jdk1.8，并上传到/usr/local/src目录，然后解压

```shell
$ sudo cd /usr/local/src
$ sudo tar zxf jdk-8u291-linux-x64.tar.gz -C /usr/local/src
$ sudo ls
jdk1.8.0_291  jdk-8u291-linux-x64.tar.gz
```

### 三、创建Dockerfile

先在/usr/local目录下创建jdk目录，并将/usr/local/src下的jdk-8u291-linux-x64.tar.gz复制到/usr/local/jdk目录下，然后创建Dockerfile文件

```shell
$ sudo mkdir /usr/local/jdk
$ sudo cd /usr/local/jdk
$ sudo cp ../src/jdk-8u291-linux-x64.tar.gz ./
$ sudo ls
jdk-8u291-linux-x64.tar.gz
$ sudo vi Dockerfile

FROM centos:centos7
MAINTAINER tom
RUN mkdir /usr/local/jdk
WORKDIR /usr/local/jdk
ADD jdk-8u291-linux-x64.tar.gz /usr/local/jdk

ENV JAVA_HOME /usr/local/jdk/jdk1.8.0_291
ENV JRE_HOME /usr/local/jdk/jdk1.8.0_291/jre
ENV PATH $JAVA_HOME/bin:$PATH
```

### 四、使用Dockerfile构建jdk1.8镜像

```shell
$ sudo docker build -t jdk8 .
Sending build context to Docker daemon  144.9MB
Step 1/8 : FROM centos:centos7
centos7: Pulling from library/centos
Digest: sha256:0f4ec88e21daf75124b8a9e5ca03c37a5e937e0e108a255d890492430789b60e
Status: Downloaded newer image for centos:centos7
 ---> 8652b9f0cb4c
Step 2/8 : MAINTAINER tom
 ---> Running in 8b9ec5e744fc
Removing intermediate container 8b9ec5e744fc
 ---> 6bfac971caf6
Step 3/8 : RUN mkdir /usr/local/jdk
 ---> Running in 70147911c16c
Removing intermediate container 70147911c16c
 ---> 04e20420c47f
Step 4/8 : WORKDIR /usr/local/jdk
 ---> Running in 09eb8f554a0d
Removing intermediate container 09eb8f554a0d
 ---> 1e3969375470
Step 5/8 : ADD jdk-8u291-linux-x64.tar.gz /usr/local/jdk
 ---> 43215bd8c5cf
Step 6/8 : ENV JAVA_HOME /usr/local/jdk/jdk1.8.0_291
 ---> Running in e7f96207c557
Removing intermediate container e7f96207c557
 ---> 6e5e459ab6a8
Step 7/8 : ENV JRE_HOME /usr/local/jdk/jdk1.8.0_291/jre
 ---> Running in b6e65916850c
Removing intermediate container b6e65916850c
 ---> a1c5c519cb84
Step 8/8 : ENV PATH $JAVA_HOME/bin:$PATH
 ---> Running in 1f1c01793249
Removing intermediate container 1f1c01793249
 ---> ccfdfab786fb
Successfully built ccfdfab786fb
Successfully tagged jdk8:latest
```

### 五、在镜像仓库中查看是否构建成功

```
$ sudo  docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
jdk8                latest              ccfdfab786fb        About a minute ago   564MB
```

### 六、启动jdk容器

```shell
$ sudo docker run -di --name=jdk1.8 jdk1.8
e04f670691cd301b28fb56c25b12eae4851f583fa7abac367164a0ee68ad7241
```

### 七、进入jdk容器，查看是否安装正确（即查看安装之后的目录）

```shell
$ sudo docker exec -it jdk1.8 /bin/bash
[root@7d539233feb0 jdk]# pwd
/usr/local/jdk
[root@7d539233feb0 jdk]# ls
jdk1.8.0_291
```