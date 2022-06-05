引入`gateway`jar包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

```



- Spring Cloud Gateway 不使用 Web 作为服务器，而是 **使用 WebFlux 作为服务器**，Gateway 项目已经依赖了 `starter-webflux`，所以这里 **千万不要依赖 `starter-web`**



或者移除`starter-web`下面两个jar包

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!--gateway使用的是webflux，与webmvc和tomcat冲突 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
	</exclusions>
</dependency>
```



由于过滤器等功能依然需要 Servlet 支持，故这里还需要依赖 `javax.servlet:javax.servlet-api`

```xml

<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
</dependency>
```



### 服务转发配置

yaml配置

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名称进行路由
      #路由设置
      routes:
        - id: test_route
          uri: lb://service-test #使用注册中心后，可以直接配置服务器名
          predicates:
            - Path=/test/**
```



官网Gateway文档中还有许多配置	
	https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.3.RELEASE/reference/html/#configuring-route-predicate-factories-and-gateway-filter-factories



### 服务降级配置



加入服务熔断jar包

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

熔断器配置

```yaml
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
          thread:
            timeoutInMilliseconds: 3000 #超时事件 3秒
```

路由配置

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名称进行路由
      #路由设置
      routes:
        - id: test_route
          uri: lb://service-test #使用注册中心后，可以直接配置服务器名
          predicates:
            - Path=/test/**		#所有请求网关 第一个url路径为test的请求，会转发到 service-test 这个服务上
          filters:
            - StripPrefix=1		#截取请求路径第一个url  localhost:8888/test/hello
            - name: Hystrix
              args:
                name: testFallback
                fallbackUri: forward:/testfallback #服务熔断后 转发url
```

服务熔断后，会转发到相应的url进行处理

```java
/**
 * 响应超时熔断处理器
 */
@Slf4j
@RestController
public class FallbackController {

    @RequestMapping("/testfallback")
    public Map<String,String> testFallback(){
        log.error("test服务降级操作...");
        Map<String,String> map = new HashMap<>(3);
        map.put("resultCode","fail");
        map.put("resultMessage","服务异常");
        map.put("resultObj","null");
        return map;
    }
}
```


