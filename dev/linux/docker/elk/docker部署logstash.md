# 1、拉取镜像

​	logstash版本与 elasticsearch、kibana版本保持一致

```shell
docker pull logstash:7.12.1
```

# 2、创建配置文件目录

- 指定目录进行创建config和pipeline文件

```shell
mkdir -p /usr/local/logstash/config  /usr/local/logstash/pipeline/
```

# 3、配置yml文件

​	在config文件进行新建logstash.yml和pipelines.yml文件

## logstash.yml配置内容

```yml
config:
  reload:
    automatic: true
    interval: 3s
xpack:  
  management:
    enabled: false
  monitoring:
    enabled: false
```

## pipelines.yml配置内容

```yaml
- pipeline.id: logstash_dev
  path.config: "/usr/share/logstash/pipeline/logstash_dev.conf"
```



## 配置conf文件

​		在pipeline文件夹中配置conf文件，指定一个名称进行配置，比如：logstash_dev.conf表示开发环境的logs有多个环境可以创建多个文件进行指定配置同步到容器执行即可。

```conf
input {
	tcp {
		mode => "server"
		host => "0.0.0.0"
		port => 5047
		codec => json_lines  
	}
}
filter{

}
output {
	elasticsearch {
		hosts => ["192.168.154.98:9200"]
		index => "logstash-dev-%{+YYYY.MM.dd}"
	}    
	stdout {
		codec => rubydebug
	}
} 
```

# 4、运行容器

- 命令操作,在运行有转义问题，建议把去掉，整成一行命令脚本进行执行

```
docker run -d -it --restart=always --privileged=true --name=logstash -p 5047:5047 -p 9600:9600 -v /usr/local/logstash/pipeline/:/usr/share/logstash/pipeline/ -v /usr/local/logstash/config/:/usr/share/logstash/config/ logstash:7.12.1
```

