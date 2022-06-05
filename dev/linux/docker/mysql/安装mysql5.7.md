### 1.拉取mysql镜像文件

拉取最新版本(也可以指定版本)

```shell
docker pull mysql
```

检查本地镜像文件

```shell
docker images
```

2.创建配置文件
创建配置文件存放位置 和数据映射位置

```shell
mkdir -p /usr/local/lib/mysql/conf /usr/local/lib/mysql/data /var/log/mysql/
```

创建编辑配置文件

```shell
vi /usr/local/lib/mysql/conf/my.conf
```

my.conf配置文件内容如下

```conf
[mysqld]
user=mysql
character-set-server=utf8
default_authentication_plugin=mysql_native_password

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
```

3.启动容器

```
docker run -d -p 3306:3306 \
--name mysql \
--restart=always \
--privileged=true \
-v /usr/local/lib/mysql/conf/my.conf:/etc/my.cof \
-v /var/log/mysql/:/logs \
-v /usr/local/lib/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
 mysql:5.7
```

参数说明:

```
-d 	后台运行容器
-p 3306:3306 	指定端口映射(主机(宿主)端口:容器端口)
--restart=always 	开机启动
--privileged=true 	提升容器内权限
--name 	为容器指定一个名称
-e  	设置环境变量
	MYSQL_ROOT_PASSWORD=123456 	初始密码
-v /usr/local/lib/mysql/conf/my.conf:/etc/my.cof 映射配置文件
-v /usr/local/lib/mysql/data:/var/lib/mysql 映射数据目录
-v /var/log/mysql/:/logs 映射日志
mysql 	镜像名称
```

