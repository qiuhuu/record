### 在 `pom.xml` 中增加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### 在 Application 中增加 `@EnableHystrix` 注解

```java
package com.qiuhuu.cloud.test;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.ComponentScan;

@SpringCloudApplication
@EnableFeignClients
@ComponentScan("com.qiuhuu.cloud")
@EnableHystrix
public class TestApplication {

    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }

}

```



在方法上使用@HystrixCommand 注解

```java
/**
 * 自定义方法的超时熔断时间
 * fallbackMethod 熔断后进入的方法
 */
@HystrixCommand(fallbackMethod = "testHystrixFallback",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "5000")
})
public String  testHystrix(){
    System.out.println("hello");
}

public String  testHystrixFallback(){
    System.out.println("方法异常");
}
```



fallbackMethod：当方法发生异常，或者超出`@HystrixProperty`设置的超时时间，会进入到testHystrixFallback方法里

commandProperties：Hystrix属性值

​	`execution.isolation.thread.timeoutInMilliseconds`：服务访问超时时执行隔离线程中断



注： 所有`commandProperties` 参数 都在 `HystrixPropertiesManager.class` 中，可以自己去阅读