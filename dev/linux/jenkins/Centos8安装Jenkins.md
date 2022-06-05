# Jenkins简介

​		Jenkins是一个开源的软件项目，是基于java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。

1. 持续的软件版本发布/测试项目
2. 监控外部调用执行的工作

# Jenkins环境准备

1. 安装jenkins前确保您的电脑已经配置好JDK

	#### 安装oracle JDK

	下载地址 https://www.oracle.com/java/technologies/javase-jdk11-downloads.html

	```shell
	# 自行下载后进入安装包存放位置
	rpm -ivh jdk-8u181-linux-x64.rpm
	#或者 tar包 安装方法
	tar xf jdk-8u251-linux-x64.tar.gz -C /usr/local/
	mv /usr/local/jdk-8u251/ /usr/local/jdk
	  
	vim /etc/profile
	export JAVA_HOME=/usr/local/jdk
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export  PATH=${JAVA_HOME}/bin:$PATH
	  
	source /etc/profile
	 
	# 验证安装
	[root@node2 ~]# java -version
	java version "1.8.0_251"
	Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
	Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
	```

2. 下载好jenkins安装包

	jenkins下载地址：https://www.jenkins.io/zh/download/

	选择rpm文件

3. Maven

	maven下载地址：https://maven.apache.org/download.cgi

4. Centos8

**注意:** 如果将Jenkins作为Docker 容器运行，JDK不是必需的



# Jenkins部署方式

## 安装Jenkins

```shell
rpm -ivh jenkins-2.235.1-1.1.noarch.rpm
```

## 修改配置文件

开放jenkins端口

```shell
firewall-cmd --add-port=8080/tcp --permanent
```

设置访问用户

```shell
vim /etc/sysconfig/jenkin
JENKINS_USER="root"
```

手动安装的jdk要在 jenkins 配置文件中指一下: /etc/init.d/jenkins 

```shell
vim /etc/init.d/jenkins
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/bin/java
/usr/local/jdk/bin/java"           # 新添加jdk路径
```

修改插件库源需要先启动下jenkins才会生成配置文件

```shell
vi /var/lib/jenkins/hudson.model.UpdateCenter.xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
   <--! <url>https://updates.jenkins.io/update-center.json</url> -->
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
```

修改升级地址

```shell
sed -i 's@http://updates.jenkins-ci.org/download/@https://mirrors.tuna.tsinghua.edu.cn/jenkins/@g' /var/lib/jenkins/updates/default.json

sed -i 's@http://www.google.com/@http://www.baidu.com/@g' /var/lib/jenkins/updates/default.json
```

启动并设置开机启动

```shell
systemctl restart jenkins && systemctl enable jenkins
```

## 浏览器访问

- 访问: http://server_ip:8080
- 查看 admin 默认密码: cat /var/lib/jenkins/secrets/initialAdminPassword
- 选择默认插件 进行安装