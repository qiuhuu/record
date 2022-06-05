# 1、调用数据压缩优化

加入它的依赖：

```xml
<dependency>
	<groupId>io.github.openfeign</groupId>
	<artifactId>feign-okhttp</artifactId>
</dependency>
复制代码
```

开启服务端配置：

```yaml
server:
  port:8888
  compression:
    enabled:true
    min-response-size:1024
    mime-types:["text/html","text/xml","application/xml","application/json","application/octet-stream"]
```

开启客户端配置：

```yaml
feign:
  httpclient:
    enabled:false
  okhttp:
    enabled:true
```

经过这些压缩之后，我们的接口平均响应时间，直接从5-6秒降低到了2-3秒，优化效果非常显著。

