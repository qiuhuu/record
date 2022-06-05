# docker安装redis步骤

## 一、拉取redis版本

### 	1、查到响应的redis版本

​			可以去[docker hub](https://hub.docker.com/_/redis?tab=tags)中 查询 相应的版本

### 	2、使用docker拉取redis镜像

```shell
docker pull redis:镜像版本号
```

 		不加镜像版本号也可以直接拉取  latest 版本

```shell
docker pull redis
```

​	如果权限不足，用sudo 提权拉取

## 二、准备redis的配置文件

 	到redis官方中下载redis使用的配置文件

  http://www.redis.cn/download.html

​		主要需要的是redis.conf文件

四、配置redis.conf配置文件
修改redis.conf配置文件：
主要配置的如下：  (6.6版本)

```conf
bind 127.0.0.1 #注释掉这部分，使redis可以外部访问    69行
daemonize no #用守护线程的方式启动				222行
requirepass 你的密码#给redis设置密码				786行
appendonly yes #redis持久化　　默认是no			1059行
tcp-keepalive 300 #防止出现远程主机强迫关闭了一个现有的连接的错误 默认是300  在130行
```

vi模式里 输入 :set nu 可以显示行号





## 三、创建本地与docker映射的目录

创建本地存放redis配置的位置/usr/local/lib/redis

```shell
mkdir /usr/local/lib/redis  #没有权限sudu提权
mkdir /usr/local/lib/redis/data
```

把配置文件拷贝到 /usr/local/lib/redis



## 四、启动docker redis

```shell
docker run --restart=always -p 6379:6379 --name redis -v /usr/local/lib/redis/redis.conf:/etc/redis/redis.conf  -v /usr/local/lib/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
```



-p 6379:6379：把容器内的6379端口映射到宿主机6379端口
-v /data/redis/redis.conf:/etc/redis/redis.conf：把宿主机配置好的redis.conf放到容器内的这个位置中
-v /data/redis/data:/data：把redis持久化的数据在宿主机内显示，做数据备份

-d reids 后台运行容器，并返回容器ID；

redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动
–appendonly yes：redis启动后数据持久化





## 五、相关问题

### 1、redis无法访问

​	1）、请检查防火墙是否开放 6379端口

​	2）、如果配置文件中`protected-mode yes` ，但没有设置密码或者绑定ip

​		Redis运行在保护模式，因为启用了保护模式，没有指定绑定地址，也没有向客户端请求认证密码。在这种模式下，只接受来自loopback接口的连接。如果你想从外部计算机连接到Redis，你可以采用以下解决方案之一:

​			1)只要禁用保护模式发送命令'CONFIG SET protected-mode no'从loopback接口连接到服务器运行的同一主机上的Redis，但是确保Redis不是公共访问从互联网，如果你这样做。使用CONFIG REWRITE使此更改永久性。

​			2)或者你可以通过编辑Redis配置文件来禁用保护模式，并将保护模式选项设置为"no"，然后重启服务器。

​			3)如果你手动启动服务器只是为了测试，用"protected-mode no"选项重启它。

​			4)设置绑定地址或认证密码。

注意:为了让服务器开始接受来自外部的连接，您只需要做上面的一件事。

