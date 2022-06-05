### `/actuator/gateway/globalfilters`

​	要检索应用中所有路由的全局过滤器，`GET`发出请求`/actuator/gateway/globalfilters`，会得到下面内容

```json
{
    "org.springframework.cloud.gateway.filter.GatewayMetricsFilter@36cf16a6": 0,
    "org.springframework.cloud.gateway.filter.NettyRoutingFilter@5c1dd18": 2147483647,
    "org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@6282f1eb": -2147482648,
    "org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@2a6dbb7c": -1,
    "org.springframework.cloud.gateway.filter.LoadBalancerClientFilter@3573e19d": 10100,
    "org.springframework.cloud.gateway.filter.ForwardPathFilter@377e90b0": 0,
    "org.springframework.cloud.gateway.filter.ForwardRoutingFilter@708f7386": 2147483647,
    "org.springframework.cloud.gateway.filter.RemoveCachedBodyFilter@5178345d": -2147483648,
    "org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@7b2d58e6": 10000,
    "org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@27b490de": 2147483646
}
```



### `/actuator/gateway/routes`

在配置文件中开启

```properties
management.endpoint.gateway.enabled=true
```

​	可以获取Gateway中每个路由更多详细信息，可以查看与每个路由关联的谓词和过滤器以及任何可用的配置。以下示例进行配置`/actuator/gateway/routes`

```json
[
  {
    "predicate": "(Hosts: [**.addrequestheader.org] && Paths: [/headers], match trailing slash: true)",
    "route_id": "add_request_header_test",
    "filters": [
      "[[AddResponseHeader X-Response-Default-Foo = 'Default-Bar'], order = 1]",
      "[[AddRequestHeader X-Request-Foo = 'Bar'], order = 1]",
      "[[PrefixPath prefix = '/httpbin'], order = 2]"
    ],
    "uri": "lb://testservice",
    "order": 0
  }
]
```

默认情况下启用此功能。要禁用它，设置以下属性：

application.properties

```properties
spring.cloud.gateway.actuator.verbose.enabled=false
```



### `/actuator/gateway/routefilters`

​	路由过滤器：	

​	要检索应用于路线的`GatewayFilter`工厂，`GET`请求`/actuator/gateway/routefilters`。产生的响应类似于以下内容：

```
{ 
  “ [ AddRequestHeaderGatewayFilterFactory @ 570ed9c configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]”：null，
  “ [ SecureHeadersGatewayFilterFactory @ fceab5d configClass = Object]”：null，
  “ [[ SaveSessionGatewayFilterFactory @ 4449b273 configClass = Object]”：null 
}
```

​	该响应包含`GatewayFilter`应用于任何特定route的工厂的详细信息。对于每个工厂，都有一个对应对象的字符串表示形式（例如`[SecureHeadersGatewayFilterFactory@fceab5d configClass = Object]`）。请注意，该`null`值是由于端点控制器的实现不完整所致，因为它试图设置对象在过滤器链中的顺序，而该顺序不适用于`GatewayFilter`工厂对象。



### `/actuator/gateway/refresh`

​	刷新路由缓存

​	要清除路由缓存，`POST`发送请求`/actuator/gateway/refresh`。该请求返回200，但没有响应正文。



### `/actuator/gateway/routes`

​	检索网关中定义的路由

要检索网关中定义的路由，`GET`发出请求`/actuator/gateway/routes`。产生的响应类似于以下内容：

```json
[{ 
  “ route_id”：“ first_route”，
  “ route_object”：{ 
    “ predicate”：“ org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory $$ Lambda $ 432/1736826640 @ 1e9d7e7d ”，
    “ filters”：[[ 
      OrderedGatewayFilter {delegate = org.springframework.cloud.gateway.filter.factory.PreserveHostHeaderGatewayFilterFactory $$ Lambda $ 436/674480275 @ 6631ef72 ，order = 0}“ 
    ] 
  }，
  ” order“：0 
}，
{ 
  ” route_id“：” second_route“，
  ” route_object“：{ 
    ” predicate“：” org.springframework.cloud.gateway.handler.predicate。PathRoutePredicateFactory $$ Lambda $ 432/1736826640 @ cd8d298 “，
    “过滤器”：[] 
  }，
  “订单”：0 
}]
```

该响应包含网关中定义的所有路由的详细信息。下表描述了响应的每个元素的结构（每个元素都是一条路线）：

| 路径                     | 类型   | 描述                              |
| :----------------------- | :----- | :-------------------------------- |
| `route_id`               | String | 路由ID。                          |
| `route_object.predicate` | Object | 路由谓词。                        |
| `route_object.filters`   | Array  | 该`GatewayFilter`工厂使用的路由。 |
| `order`                  | Number | 路线顺序。                        |



### `/actuator/gateway/routes/{id}`

​	要检索有关一条路线的信息，`GET`发出请求`/actuator/gateway/routes/{id}`（例如`/actuator/gateway/routes/first_route`）。产生的响应类似于以下内容：

```
{
  "id": "first_route",
  "predicates": [{
    "name": "Path",
    "args": {"_genkey_0":"/first"}
  }],
  "filters": [],
  "uri": "https://www.uri-destination.org",
  "order": 0
}]
```

下表描述了响应的结构：

| 路径         | 类型   | 描述                                                   |
| :----------- | :----- | :----------------------------------------------------- |
| `id`         | String | 路由ID。                                               |
| `predicates` | Array  | 路由谓词的集合。每个项目都定义给定谓词的名称和自变量。 |
| `filters`    | Array  | 应用于路线的过滤器集合。                               |
| `uri`        | String | 路由的目标URI。                                        |
| `order`      | Number | 路线顺序。                                             |

### 

### `/gateway/routes/{id_route_to_create}`

创建和删除特定路线

​	要创建路由，`POST`发出请求`/gateway/routes/{id_route_to_create}`使用JSON主体发出请求，以指定路由的字段（请参阅 `/actuator/gateway/routes/{id}` 的响应内容）。

要删除路线，`DELETE`发出请求`/gateway/routes/{id_route_to_delete}`。

### 所有端点的列表

下表列出了Spring Cloud Gateway执行器端点（请注意，每个端点都`/actuator/gateway`作为基本路径）：

| ID              | HTTP方法 | 描述                                          |
| :-------------- | :------- | :-------------------------------------------- |
| `globalfilters` | Get      | 显示应用于路由的全局过滤器列表。              |
| `routefilters`  | Get      | 显示`GatewayFilter`应用于特定路线的工厂列表。 |
| `refresh`       | Post     | 清除路由缓存。                                |
| `routes`        | Get      | 显示网关中定义的路由列表。                    |
| `routes/{id}`   | Get      | 显示有关特定路线的信息。                      |
| `routes/{id}`   | Post     | 将新路由添加到网关。                          |
| `routes/{id}`   | Delete   | 从网关删除现有路由。                          |