---
title: 网关调优指导书
tags: spring-cloud
date: 2017-12-21
---

由于最近在使用Spring Cloud的Zuul网关的过程中，发现超时的可能性很多，出于性能的调优，所有想通过测试，了解一些参数的作用。在文章最后贴上推荐方案。

<!--more-->

先看一个问题：

`execution.isolation.thread.timeoutInMilliseconds`

到达当前时间后，会触发熔断，调用fallback方法，如果不存在fallback方法，会报错误

```json
{
    "code": "500",
    "message": "[Internal Server Error]getStores timed-out and fallback failed.",
    "body": null
}
```

结果：

```Txt
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.client.ClientException: null
Caused by: java.net.SocketTimeoutException: Read timed out
```

> 说明：下面zuul是网关配置，service是网关代理下的一个服务

### Case1

zuul，延时3s

```
ribbon:
  ConnectTimeout: 2000
  # 请按实际情况配置
  ReadTimeout: 1000
```

service，延时2s

第一次访问的时候

```txt
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.client.ClientException: Load balancer does not have available server for client: hystrix-example
	
```

第二次访问的时候

```
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.hystrix.exception.HystrixRuntimeException: hystrix-example timed-out and no fallback available.
Caused by: java.util.concurrent.TimeoutException: null

###result
{
    "code": "500",
    "message": "[Internal Server Error]TIMEOUT",
    "body": null
}
```

### Case2

zuul，延时3s

```
ribbon:
  ConnectTimeout: 4000
  # 请按实际情况配置
  ReadTimeout: 1000
```

service，延时2s

```
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.hystrix.exception.HystrixRuntimeException: hystrix-example timed-out and no fallback available.
Caused by: java.util.concurrent.TimeoutException: null

###result
{
    "code": "500",
    "message": "[Internal Server Error]TIMEOUT",
    "body": null
}
```

### Case3

zuul，延时3s

```
ribbon:
  ConnectTimeout: 4000
  # 请按实际情况配置
  ReadTimeout: 1000
```

service，延时0.5s

正常

### Case4

zuul，延时3s

```
ribbon:
  ConnectTimeout: 4000
  # 请按实际情况配置
  ReadTimeout: 1000
```

service，延时2s

```
hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 60000
```

```
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.hystrix.exception.HystrixRuntimeException: hystrix-example timed-out and no fallback available.
Caused by: java.util.concurrent.TimeoutException: null

###result
{
    "code": "500",
    "message": "[Internal Server Error]TIMEOUT",
    "body": null
}
```

结论：

- hystrix超时时间在配置文件中配置时无效的

```
hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 60000
```

- hystrix默认超时时间四1s，如果服务执行时间超过1s就会进行熔断，如果没有fallback，就会导致TIMEOUT错误

### Case5

zuul，延时3s

```
ribbon:
  ConnectTimeout: 500
  # 请按实际情况配置
  ReadTimeout: 2000
```

service，延时0.5s

正常

### Case6

zuul，延时3s

```
ribbon:
  ConnectTimeout: 400
  # 请按实际情况配置
  ReadTimeout: 400
```

service，延时0.5s

```
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.client.ClientException: null
Caused by: java.lang.RuntimeException: java.net.SocketTimeoutException: Read timed out
Caused by: java.net.SocketTimeoutException: Read timed out
####reuslt
{
    "code": "500",
    "message": "[Internal Server Error]GENERAL",
    "body": null
}
```

### Case7

zuul，延时3s

```
ribbon:
  ConnectTimeout: 400
  # 请按实际情况配置
  ReadTimeout: 600
```

service，延时0.5s

正常

结论：

- ribbon.readtimeout超时会导致`SocketTimeoutException: Read timed out`问题。

### Case8

zuul，延时3s

```
ribbon:
  ConnectTimeout: 1000
  # 请按实际情况配置
  ReadTimeout: 20000
zuul:
  host:
    # 连接超时时间
    connect-timeout-millis: 3000
    # 响应超时时间
    socket-timeout-millis: 100
```

service，延时0.5s

正常

结论：socket-timeout-millis对请求时间没有影响

### Case9

zuul，延时3s

```
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            # 请按实际情况设置配置
            timeoutInMilliseconds: 600

ribbon:
  ConnectTimeout: 1000
  # 请按实际情况配置
  ReadTimeout: 20000
```

service，延时0.5s

正常

### Case10

zuul，延时3s

```
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            # 请按实际情况设置配置
            timeoutInMilliseconds: 400

ribbon:
  ConnectTimeout: 1000
  # 请按实际情况配置
  ReadTimeout: 20000
```

service，延时0.5s

```
com.netflix.zuul.exception.ZuulException: Forwarding error
Caused by: com.netflix.hystrix.exception.HystrixRuntimeException: hystrix-example timed-out and no fallback available.
Caused by: java.util.concurrent.TimeoutException: null

###result
{
    "code": "500",
    "message": "[Internal Server Error]TIMEOUT",
    "body": null
}
```

结论：

- Hystrix的超时时间是对次节点的请求时间的进行熔断

zuul，延时3s

service，延时0.5s

```
Caused by: com.netflix.hystrix.exception.HystrixRuntimeException: hystrix-example could not be queued for execution and no fallback available.
error 500 : REJECTED_THREAD_EXECUTION
```

### 总结：

- Hystrix的超时配置是服务调用其他服务时候，有效，对服务自身是没有作用。e.g. A->B，是对A请求B这个请求有效，对访问A本身的请求是没有作用的。
- 网关在启动初期，会存在不稳定，甚至存在马上熔断的可能，但在之后，会恢复正常水平。
- `com.netflix.client.ClientException: Load balancer does not have available server for client: hystrix-example`是服务启动还没完全，或者，如果使用了`ribbon.eureka.eabled=false`也会出现这个问题。
- 服务启动时候，推荐网关最后启动

推荐使用方案：

```yaml
zuul:
  okhttp:
    enabled: true # 使用okhttp方式请求，正常来说okhttp比较速度快一点
  semaphore:
    max-semaphores: 500 # 并发处理数，值越大越好，但到到达一个临界点之后，就不会提高响应速度了
  host:
    socket-timeout-millis: 30000 # socket超时时间，如果使用service-id方式是不用配置的
    connect-timeout-millis: 30000 # 连接时间semaphores
    max-total-connections: 5000  # 最大连接数，值越大越好，但到到达一个临界点之后，就不会提高响应速度了
    max-per-route-connections: 5 # 每个router最大连接数，降低请求时间，越小越好，但达到一定层级就没用了

hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 30000 # Hystrix超时时间
          strategy: THREAD

ribbon:
  ReadTimeout: 20000 # 处理时间
  ConnectTimeout: 20000 # 连接时间
  MaxAutoRetries: 0 #最大自动重试次数
  MaxAutoRetriesNextServer: 1 # 换实例重试次数
  MaxTotalHttpConnections: 2000 # 最大http连接数，越大越好，但到到达一个临界点之后，就不会提高响应速度了
  MaxConnectionsPerHost: 1000 # 每个host连接数
```

注意说明：

> `timeout-in-milliseconds`这样编写是不会生效的，需要改为`timeoutInMilliseconds`，起初认为是Spring的BUG，之后，发现由于default是一个key，是一个Map类型，依照源码中使用的是`timeoutInMilliseconds`，所以必须`timeoutInMilliseconds`。

### 拓展学习：

[http://www.spring4all.com/article/351](http://www.spring4all.com/article/351)

[http://m635674608.iteye.com/blog/2389666](http://m635674608.iteye.com/blog/2389666)

[http://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#how-to-configure-hystrix-thread-pools](http://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#how-to-configure-hystrix-thread-pools)

[http://tietang.wang/2016/02/25/hystrix/Hystrix%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3/](http://tietang.wang/2016/02/25/hystrix/Hystrix%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3/)

[https://github.com/spring-cloud/spring-cloud-netflix/issues/327](https://github.com/spring-cloud/spring-cloud-netflix/issues/327)

[zuul性能测试](https://www.cnkirito.moe/2017/04/08/Zuul%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95/)