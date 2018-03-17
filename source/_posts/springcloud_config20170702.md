---
title: Spring Cloud Config
tags: spring-cloud
date: 2017-07-02
---
config是Spring Cloud中的配置中心，在正式场景中，存在修改配置的情况，每次配置的修改都要进行重新打包，这是非常麻烦的一件事，可能还伴随着其他问题的引发。而config就可以将一些与启动无关的配置进行动态修改，并生效。以前要数据库进行配置的，现在也可以在config中完成。

<!-- more -->

### 快速入门

### Config服务端

```xml
 		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>    
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
    </dependencies>
```

`bootstrap.yml`

```yaml
# Spring properties
spring:
  application:
     name: config-service  # Service registers under this name
  cloud:
    config:
      server:
        git:
          uri: https://github.com/zoeminghong/spring-cloud-demo.git
          search-paths: config-repo #文件搜索路径
          username: username #账号
          password: password #密码
      name: appzone #application name
      label: master #分支
#  profiles:
#    active: native #当不使用git时，可以设置为native，获取config服务下main/java/resources本地配置


# Map the error path to error template (for Thymeleaf)
error:
  path=/error

# Discovery Server Access
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://root:123456@${eureka.instance.hostname}:8761/eureka/
# 注册到eureka服务，账号：密码@地址
# HTTP Server
server:
  port: 7001   # HTTP (Tomcat) port
```

`Main.java`

```java
@EnableConfigServer
@SpringBootApplication
@EnableDiscoveryClient
public class ConfigApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(ConfigApplication.class).web(true).run(args);
    }
}
```

### Config客户端

pom配置是与服务端是一样的

```yaml
spring:
  cloud:
#  config的相关配置
    config:
      label: master
      profile: dev
      uri: http://localhost:7001/ #config Service address
      name: appzone #application name
      discovery:
        enabled: true #whether enable service-id
        service-id: CONFIG-SERVICE #config service application name on the eureka
      username: root # regisit centre's username
      password: 123456 # regisit centre's password
#      fail-fast: true # 没有连接配置服务端时直接启动失败
# Spring properties
  application:
     name: order-service  # Service registers under this name
  freemarker:
    enabled: false           # Ignore Eureka dashboard FreeMarker templates
  thymeleaf:
    cache: false

# Discovery Server Access
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://root:123456@${eureka.instance.hostname}:8761/eureka/

# HTTP Server
server:
  port: 2222   # HTTP (Tomcat) port
```

`main.java`

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(OrderApplication.class).web(true).run(args);
    }
}
```

### 深入学习

### 刷新配置

git配置目录下的文件发生更改时，需要更新通知到服务，使用@RefreshScope可以帮助实现配置的刷新。

实现方式：

在指定的配置类下使用@RefreshScope，如若git配置发生变化，使用http://相应服务地址/refresh ,(POST)。

也可以：

**重启服务**在application.yml中启用`endpoints.restart.enabled=true`，调用http://相应服务地址/restart ,(POST)服务。

**注**-往往存在一些场景，refresh是不会生效的，因而，使用restart时比较保险的操作，但restart耗时比较长。故建议，在没有特殊的处理的配置类中使用@RefreshScope来实现refresh，存在比较复杂，且要求比较高的配置项，还是使用restart比较靠谱。

### 模式的匹配

```yaml
spring:
cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
```

当不存在pattern时，{application}/{profile}则根据key来决定，例如simple中，匹配的是`simple/*`，如local中，匹配的是`local*/*`

本地存储路径控制：

在使用的config服务的时候，其会clone一份缓存到本地，如果你要指定路径可以使用`spring.cloud.config.server.git.basedir`

使用本地加载配置文件：

需要配置：`spring.cloud.config.server.native.searchLocations`跟`spring.profiles.active=native`。
路径配置格式：`classpath:/, classpath:/config,file:./, file:./config`。

基于文件的资源库：

在基于文件的资源库中(i.e. git, svn and native)，这样的文件名`application*`命名的资源在所有的客户端都是共享的(如 application.properties, application.yml, application-*.properties,etc.)。

### 加密与解密

如果远程属性包含加密内容(以{cipher}开头),这些值将在通过HTTP传递到客户端之前被解密。

**实现方式**下载解压[JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)，并复制至`JDK/jre/lib/security`文件夹下，Maven依赖”`org.springframework.security:spring-security-rsa`”。

### 环境配置

config相关配置需要在bootstrap.yml中进行配置，在实际开发中存在调试环境，开发环境，测试环境，线上环境等场景，因而，对bootstrap.yml进行配置环境化配置是很必须的。

可以bootstrap-[environment].yml，默认是会读取bootstrap.yml和bootstrap-default.yml中的配置。若需要读取其他环境的配置，可在bootstrap.yml中设置

```yaml
#spring环境和config中的配置都会使用该环境的配置
spring:
	profile:
		active: environment
```

只是想更改config中的环境：

```yaml
spring:
	cloud:
		config:
			profile: environment
```

### Tips

如果config的客户端需要使用service-id这种负载均衡的方式获取config服务端的配置信息，需要注意将**注册中心的信息和config服务的信息**都写于bootstrap.yml下，

否则可能存在找不到config服务。

[config中文文档](http://www.cnblogs.com/powercto/p/6726991.html)