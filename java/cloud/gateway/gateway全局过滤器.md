### 全局过滤器

实现 `GlobalFilter`, `Ordered` 接口并在类上增加 `@Component` 注解就可以使用过滤功能了，非常简单方便

当请求与路由匹配时，过滤Web处理程序会将的所有实例`GlobalFilter`和所有特定`GatewayFilter`于路由的实例添加到过滤器链中。该组合的过滤器链按`org.springframework.core.Ordered`接口排序，可以通过实现该`getOrder()`方法进行设置。

```java
@Slf4j
@Component
public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("start custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```



```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}
```

