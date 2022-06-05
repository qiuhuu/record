# 基于docker部署es集群

## 1.准备

首先准备服务器，我这里仅测试部署，所以准备了两台服务器

| ip                   | name      |
| -------------------- | --------- |
| 192.168.178.131:9300 | es-master |
| 192.168.178.133:9300 | es-node-1 |
|                      |           |

版本配置准备

| 名称          | 版本   |
| ------------- | ------ |
| Centos7       | 1810   |
| elasticsearch | 7.17.4 |
|               |        |
|               |        |



## 2.修改系统配置

1. 调高JVM线程数限制数量

   ```shell
   vi /etc/sysctl.conf
   
   vm.max_map_count=262144
   
   
   #保存后刷新配置
   sysctl -p
   ```

2. 关闭防火墙或者开放防火墙端口

   ```shell
   # 开放端口
   firewall-cmd --add-port=9300/tcp
   
   
   #关闭防火墙
   systemctl stop firewalld
   #禁用开启启动
   systemctl disable firewalld
   ```

   

## 3.es配置文件

1. 创建es配置目录，并将文件夹给予权限

   ```shell
   mkdir -p /data/elasticsearch/{data,logs,config,plugins}
   #授权
   chmod -R 777 /data/elasticsearch
   ```

   data：数据

   logs：日志

   config：配置

   plugins：插件

   如果每个节点数据配置、数据需要隔离，则在数据目录下继续区分/es1、/es2

2. 在config目录下创建文件`elasticsearch.yml`

   es01:为master节点

   ```yaml
   cluster.name: "es"
   node.name: "es01"
   node.master: true
   node.data: false
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   cluster.initial_master_nodes: ["es01"]
   network.host: 0.0.0.0
   network.publish_host: 192.168.178.131
   discovery.zen.minimum_master_nodes: 1
   discovery.zen.ping.unicast.hosts: ["192.168.178.131:9300", "192.168.178.134:9300"]
   ```

   es01:node节点

   ```yaml
   cluster.name: "es"
   node.name: "es02"
   node.master: false
   node.data: false
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   cluster.initial_master_nodes: ["es01"]
   network.host: 0.0.0.0
   network.publish_host: 192.168.178.134
   discovery.zen.minimum_master_nodes: 1
   discovery.zen.ping.unicast.hosts: ["192.168.178.131:9300", "192.168.178.134:9300"]
   ```

   ​	

   输入内容：

   ```shell
   echo "cluster.name: \"es\"
   node.name: \"es01\"
   node.master: true
   node.data: false
   http.cors.enabled: true
   http.cors.allow-origin: \"*\"
   cluster.initial_master_nodes: [\"es01\"]
   network.host: 0.0.0.0
   discovery.zen.minimum_master_nodes: 1
   discovery.zen.ping.unicast.hosts: [\"192.168.178.131:9300\", \"192.168.178.134:9300\"]">>/data/elasticsearch/config/elasticsearch.yml
   ```

   

   ```shell
   echo "cluster.name: \"es\"
   node.name: \"es02\"
   node.master: false
   node.data: true
   http.cors.enabled: true
   http.cors.allow-origin: \"*\"
   cluster.initial_master_nodes: [\"es01\"]
   network.host: 0.0.0.0
   discovery.zen.minimum_master_nodes: 1
   discovery.zen.ping.unicast.hosts: [\"192.168.178.131:9300\", \"192.168.178.134:9300\"]">>/data/elasticsearch/config/elasticsearch.yml
   ```

   

## 4.docker启动

注意：内存根据配置而定，大于本机配置会导致虚拟机无法创建

es1：同一台服务器注意修改映射目录

```
docker run -d --name=es --restart=always \
-p 9200:9200 -p 9300:9300 \
-v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-e ES_JAVA_OPTS="-Xms2048m -Xmx2048m" \
-e ES_MIN_MEM=128m \
-e ES_MAX_MEM=4096m \
elasticsearch:7.17.4
```

es2

```
docker run -d --name=es --restart=always \
-p 9200:9200 -p 9300:9300 \
-v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-e ES_JAVA_OPTS="-Xms2048m -Xmx2048m" \
-e ES_MIN_MEM=128m \
-e ES_MAX_MEM=4096m \
elasticsearch:7.17.4
```

## 5.错误问题

1. Could not create the Java Virtual Machine

   请检查映射目录是否有权限或者虚拟机配置是否超出最大值

2. ```log
   ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
   bootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
   ```

   请检查`vi /etc/sysctl.conf`是否修改`vm.max_map_count=262144`

## 6.检查是否启动成功

1. 查看日志是否启动成功
2. 访问地址：http://192.168.178.131:9200/_cat/nodes?v  查看集群节点是否正常

## 7.配置文件说明

**cluster.name: elasticsearch**

配置的集群名称，默认是elasticsearch，es服务会通过广播方式自动连接在同一网段下的es服务，通过多播方式进行通信，同一网段下可以有多个集群，通过集群名称这个属性来区分不同的集群。



**node.name: "node-01"**

当前配置所在机器的节点名，你不设置就默认随机指定一个name列表中名字，该name列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。当创建ES集群时，保证同一集群中的cluster.name名称是相同的，[node](https://so.csdn.net/so/search?q=node&spm=1001.2101.3001.7020).name节点名称是不同的；



**node.master: true**

指定该节点是否有资格被选举成为node（注意这里只是设置成有资格， 不代表该node一定就是master），默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。



**node.data: true**

指定该节点是否存储索引数据，默认为true。



**index.number_of_shards: 5**

设置默认索引分片个数，默认为5片。



**index.number_of_replicas: 1**

设置默认索引副本个数，默认为1个副本。如果采用默认设置，而你集群只配置了一台机器，那么集群的健康度为yellow，也就是所有的数据都是可用的，但是某些复制没有被分配（健康度可用 curl 'localhost:9200/_cat/health?v' 查看， 分为绿色、黄色或红色。*绿色代表一切正常，集群功能齐全，黄色意味着所有的数据都是可用的，但是某些复制没有被分配，红色则代表因为某些原因，某些数据不可用*）



**path.conf: /path/to/conf**

设置配置文件的存储路径，默认是es根目录下的config文件夹。



**path.data: /path/to/data**
设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：path.data: /path/to/data1,/path/to/data2



**path.work: /path/to/work**

设置临时文件的存储路径，默认是es根目录下的work文件夹。



**path.logs: /path/to/logs**

设置日志文件的存储路径，默认是es根目录下的logs文件夹



**path.plugins: /path/to/plugins**

设置插件的存放路径，默认是es根目录下的plugins文件夹, 插件在es里面普遍使用，用来增强原系统核心功能。



**bootstrap.mlockall: true**
设置为true来锁住内存不进行swapping。因为当jvm开始swapping时es的效率 会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。 同时也要允许elasticsearch的进程可以锁住内存，linux下启动es之前可以通过`ulimit -l unlimited`命令设置。

**network.bind_host: 192.168.0.1**
设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0，绑定这台机器的任何一个ip。


**network.publish_host: 192.168.0.1**
设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。(可以用不配)，<font color='red'>但在多个服务器的docker里面建议配上，不然查找集群时，地址会为docker内网地址</font>

**network.host: 192.168.0.1** （普遍配置为：0.0.0.0）
这个参数是用来同时设置bind_host和publish_host上面两个参数。(低版本时配置0.0.0.0，不然启动会报错。1.7.1和1.3.1版本亲测)

**transport.tcp.port: 9300**
设置节点之间交互的tcp端口，默认是9300。 如我搭建多节点，我的配置分别是9300、9302、9304

**transport.tcp.compress: true**
设置是否压缩tcp传输时的数据，默认为false，不压缩。

**http.port: 9200**
设置对外服务的http端口，默认为9200。

**http.max_content_length: 100mb**
设置内容的最大容量，默认100mb

**http.enabled: false**
是否使用http协议对外提供服务，默认为true，开启。

**gateway.type: local**
gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器等。

**gateway.recover_after_nodes: 1**
设置集群中N个节点启动时进行数据恢复，默认为1。

**gateway.recover_after_time: 5m**
设置初始化数据恢复进程的超时时间，默认是5分钟。

**gateway.expected_nodes: 2**
设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。

**cluster.routing.allocation.node_initial_primaries_recoveries: 4**
初始化数据恢复时，并发恢复线程的个数，默认为4。

**cluster.routing.allocation.node_concurrent_recoveries: 2**
添加删除节点或负载均衡时并发恢复线程的个数，默认为4。

**indices.recovery.max_size_per_sec: 0**
设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。

**indices.recovery.concurrent_streams: 5**
设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。

**discovery.zen.minimum_master_nodes: 1**
设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）

**discovery.zen.ping.timeout: 3s**
设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。

**discovery.zen.ping.multicast.enabled: false**
设置是否打开多播发现节点，默认是true。

**discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]**
设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。

例如：discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9302","127.0.0.1:9304"] 配置了三个节点

## 8.安装插件

1. 访问网站：https://www.elastic.co/guide/en/elasticsearch/plugins/7.17

2. 通过wget下载需要插件的包，例：Ingest Plugins插件下载地址：https://artifacts.elastic.co/downloads/elasticsearch-plugins/ingest-attachment/ingest-attachment-7.17.4.zip

   或者下载压缩包后上传到服务器

   ```
   #没有wget请自行下载安装
   wget https://artifacts.elastic.co/downloads/elasticsearch-plugins/ingest-attachment/ingest-attachment-7.17.4.zip
   ```

   

3. 将压缩包解压到 es的/plugins 目录下（本文中使用：/data/elasticsearch/plugins 目录）

   例：Ingest Plugins插件。解压后目录：./plugins/ingest-attachment

4. 完成以上步骤启动服务会自行加载插件