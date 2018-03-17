---
title: Spring Boot Admin最佳实践
tags: spring-boot
date: 2017-11-29
---

> 本文不进行Spring Boot Admin入门知识点说明

在`Spring Boot Actuator`中提供很多像`health`、`metrics`等实时监控接口，可以方便我们随时跟踪服务的性能指标。`Spring Boot`默认是开放这些接口提供调用的，那么就问题来了，如果这些接口公开在外网中，很容易被不法分子所利用，这肯定不是我们想要的结果。在这里我们提供一种比较好的解决方案。

<!-- more -->

- 被监控的服务配置

为被保护的http请求添加请求前缀

```yaml
management:
  context-path: /example-context #<1>
eureka:
  instance:
    status-page-url-path: ${management.context-path}/info #<2>
    health-check-url-path: ${management.context-path}/health

```

1. 添加请求前缀
2. `Spring Boot Admin`在启动的时候会去`eureka`拉去服务信息，其中`health`与`info`需要特殊处理，这两者的地址是根据`status-page-url-path`和`health-check-url-path`的值。

- `zuul`网关配置

`zuul`保护内部服务http接口

```yaml
zuul:
  ignoredPatterns: /*/example-context/** #<1>
```

1. 这里之所以不是`/example-context/**`，由于网关存在项目前缀，需要往前一级，大家可以具体场景具体配置

- `Spring Boot Admin`配置

配置监控的指标参数

```yaml
spring:
  application:
    name: monitor
  boot:
    admin:
      discovery:
        converter:
          management-context-path: /example-context # The endpoints URL prefix #<1>
      routes:
        endpoints: env,metrics,dump,jolokia,info,configprops,trace,logfile,refresh,flyway,liquibase,heapdump,loggers,auditevents,hystrix.stream
      turbine:
        clusters: default
        location: monitor

turbine:
  aggregator:
    clusterConfig: default
  appConfig: monitor-example #<2>
  clusterNameExpression: metadata['cluster']
```

1. 与应用配置的`management.context-path`相同
2. 添加需要被监控的应用`Service-Id`，以逗号分隔

讲解一下，通过创建一个请求前缀，可以在网关处使用前缀的方式将其排除，也就是外网将无法访问这些监控API，同时，内网还是可以进行加前缀的方式进行访问，为`Spring Boot Admin`提供了支持条件。`management`还支持port和ip的方式，但这两种方式有局限性，如果在同一台机器上部署多个服务，就会存在端口占用或者其他问题。这种方案还有一个好处，以上配置一旦确定以后，所有服务都不需要进行特殊化处理，可以直接使用。

### 问答：

- 问题：Full authentication is required to access this resource 

> 在被监控的服务中添加`management.security.enabled=false`

### 拓展阅读：

[spring-boot-admin-samples](https://github.com/joshiste/spring-boot-admin-samples)

[issue](https://github.com/codecentric/spring-boot-admin/issues/192)

[jolokia](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-jolokia)