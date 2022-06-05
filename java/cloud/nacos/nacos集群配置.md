# 1. Nacos支持三种部署模式

- 单机模式 - 用于测试和单机试用。
- 集群模式 - 用于生产环境，确保高可用。
- 多集群模式 - 用于多数据中心场景。

以上是官方提供的三种部署方式：单机模式对于企业来讲，仅可用于测试环境或者开发环境，不可用于生产环境；对于中小型企业来讲，生产环境选择集群模式已经足够，无需选择多集群模式，除非是做机房灾备，可以在两个机房部署两个集群。

在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力，具体的操作步骤：

- 准备一台Mysql数据库，版本要求：5.6.5+
- 初始化mysql数据库，数据库初始化文件：[nacos-mysql.sql](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)

- 修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos-config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=password
```

# 2.集群配置

## 配置集群配置文件

在nacos的解压目录nacos/conf目录下，有配置文件cluster.conf.example，请将名称修改为:cluster.conf，/nacos/conf/cluster.conf。每行配置成ip:port。（请配置3个或3个以上节点）

```conf
#it is ip
#example
192.168.0.1:8848
192.168.0.1:8849
192.168.0.2:8848
192.168.0.3:8848
```



## 配置 MySQL 数据库

生产使用建议至少主备模式，或者采用高可用数据库。

初始化语句和上面单机版保持一致。

## application.properties 配置

```properties
# spring

server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848

nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false


# metrics for prometheus
#management.endpoints.web.exposure.include=*

# metrics for elastic search
management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200

# metrics for influx
management.metrics.export.influx.enabled=false
#management.metrics.export.influx.db=springboot
#management.metrics.export.influx.uri=http://localhost:8086
#management.metrics.export.influx.auto-create-db=true
#management.metrics.export.influx.consistency=one
#management.metrics.export.influx.compressed=true

server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i
# default current work dir
server.tomcat.basedir=

## spring security config
### turn off security
#spring.security.enabled=false
#management.security=false
#security.basic.enabled=false
#nacos.security.ignore.urls=/**

nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**

# nacos.naming.distro.taskDispatchThreadCount=1
# nacos.naming.distro.taskDispatchPeriod=200
# nacos.naming.distro.batchSyncKeyCount=1000
# nacos.naming.distro.initDataRatio=0.9
# nacos.naming.distro.syncRetryDelay=5000
# nacos.naming.data.warmup=true
# nacos.naming.expireInstance=true

spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=password
```



## Nginx负载均衡

部署集群后，由3台集群提供管理界面，可以配置Nginx进行负载。

```conf
upstream  nacos_cluster  {
    server   192.168.0.1:8848;
    server   192.168.0.1:8849;
    server   192.168.0.2:8848;
    server   192.168.0.3:8848;
}

server{
    listen  80;
    server_name  localhost;

    location / {
        proxy_pass        http://nacos_cluster;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  
        $proxy_add_x_forwarded_for;
    }
}
```

# 3.Spring boot配置

## 配置参数

### Application.java

```java
@EnableDiscoveryClient
@SpringBootApplication
@RefreshScope
@EnableFeignClients
public class Application{
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```



bootstrap.yml

```yaml
spring:
  application:
    name: xxxx
  profiles:
    active: dev
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.0.1:8848,192.168.0.1:8849,192.168.0.2:8848,192.168.0.3:8848     #注册中心地址集群
      config:
      	server-addr: 192.168.0.1:8848,192.168.0.1:8849,192.168.0.2:8848,192.168.0.3:8848     #配置中心地址集群
```