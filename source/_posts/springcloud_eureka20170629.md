---
title: Spring Cloud Eureka
tags: spring-cloud
date: 2017-06-29
---
Eureka是一个服务的注册中心，当然，其默认也是一个客户端

<!-- more -->

### 快速入门

### Eureka服务端

`pom.xml`
```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```
`application.yml`
```yaml
spring:
  application:
     name: eureka-register
server:
  port: ${vcap.application.port:8761}   # HTTP port
eureka:
    instance:
          hostname: localhost #与本地的hosts文件配置有关
    client:
        registerWithEureka: false #表示是否将自己注册到Eureka Server，默认为true。
        fetchRegistry: false #表示是否从Eureka Server获取注册信息，默认为true。
        serviceUrl:
          defaultZone: http://localhost:${server.port}/eureka/ #设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。
    server:
        waitTimeInMsWhenSyncEmpty: 0
        enableSelfPreservation: false
```
`Application.java`

```java
@SpringBootApplication
@EnableEurekaServer//启动使用EurekaServer
public class SpringCloudEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaApplication.class, args);
	}
}
```

### Eureka客户端

`pom.xml`

同上

`bootstrap.xml`

```yaml
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/
```

`Application.java`

```java
@SpringBootApplication
@EnableEurekaClint//启动使用EurekaClient
public class SpringCloudEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaApplication.class, args);
	}
}
```

### 深入学习

### 高可用实现方案

在eureka服务端yml配置文件

```yaml
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2/eureka/
---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1/eureka/
```

在hosts文件中进行host配置

```
127.0.0.1 peer1
127.0.0.1 peer2
```

启动程序

```shell
java -jar microservice-eureka-1.0-SNAPSHOT.jar --spring.profiles.active=peer1
java -jar microservice-eureka-1.0-SNAPSHOT.jar --spring.profiles.active=peer2
```

查看eureka服务地址，是否存在两个注册地址信息

**注**-第一个服务启动的时候存在`java.net.ConnectException: Connection refused (Connection refused)`的错误信息，不用理睬，只要最后服务端口显示就表示服务起来了。出现这个错误的原因是另一个服务还没起来，无法去对方服务进行注册，如果另一个服务起来之后，不存在问题了。

**注**-hostname必须是不一致的。

### 服务提供者注册到高可用注册中心

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka1:8001/eureka/,http://eureka2:8002/eureka/
  instance:
    preferIpAddress: true
```

### 服务提供者高可用

**第一种方式**将服务部署在不同的两台服务器上，使用相同的端口

**第二种方式**将服务部署在相同的服务器上，使用不同的端口

### 消费服务调用

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka1:8001/eureka/,http://eureka2:8002/eureka/
  instance:
    preferIpAddress: true
```

如果使用了负载均衡配置情况下，会将请求分摊到两个服务提供者上。

### 健康检查和状态的路径

```yaml
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

由于存在集群和同一台服务器部署多套服务的情况，所以需要一个唯一的身份ID，默认Spring Cloud也是有一套规则
`${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}。例如 Myhost：myappname：8080 。`但如果想进行自行定规则，就可以使用上面的属性配置。

其默认是client在注册成功之后，就认为status是UP的，不会频繁的去获取心跳，当设置了该属性为true时，才会去听取心跳。并且该配置必须只能在`application.yml`下面配置，不能使用`bootstrap.yml`。

```yaml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
    healthCheckUrl: https://${eureka.hostname}/health
    homePageUrl: https://${eureka.hostname}/
```

### 添加密码

服务端：

`pom.xml`

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

`application.yml`

```Yaml
security:
  user:
    name: root
    password: 123456
```

客户端：

`application.yml`

```yaml
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://${username}：${password}@${eureka.instance.hostname}:8761/eureka/
```

### Eureka.instance.prefer-ip-address

`eureka.instance.prefer-ip-address=true`以IP的形式注册register service，false是用所在机器名（hostname）。

`eureka.instance.ip-address`自定义IP地址，其优先级比`eureka.instance.prefer-ip-address`高。

### ignored-interfaces

场景问题：

服务器上分别配置了eth0, eth1和eth2三块网卡，只有eth1的地址可供其它机器访问，eth0和eth2的 IP 无效。在这种情况下，服务注册时Eureka Client会自动选择eth0作为服务ip, 导致其它服务无法调用。

忽略指定网卡

通过上面源码分析可以得知，spring cloud肯定能配置一个网卡忽略列表。通过查文档资料得知确实存在该属性：

```yaml
spring.cloud.inetutils.ignored-interfaces[0]=eth0 # 忽略eth0, 支持正则表达式11
```

因此，第一种方案就是通过配置`application.properties`让应用忽略无效的网卡。

配置host

当网查遍历逻辑都没有找到合适ip时会走JDK的`InetAddress.getLocalHost()`。该方法会返回当前主机的hostname, 然后会根据hostname解析出对应的ip。因此第二种方案就是配置本机的hostname和`/etc/hosts`文件，直接将本机的主机名映射到有效IP地址。

手工指定IP(推荐)

添加以下配置：

```
# 指定此实例的ip
eureka.instance.ip-address=
# 注册时使用ip而不是主机名
eureka.instance.prefer-ip-address=true
```

### 服务续约

在注册完毕之后，会在每过一个间隔期，就会向注册中心续约，表示服务还活着。

```yaml
eureka:
	instance:
		lease-renewal-interval-in-secondes: 30 #服务续约任务的调用间隔时间
        lease-expiration-duration-in-seconds: 90 #定义服务失效的时间
```

也就是说，默认是每隔30秒应用向注册中心进行续约操作，当服务续约时间间隔超过90秒，注册中心就任务其服务已经失效。

### 服务消费者

获取服务清单

服务消费者会在启动的使用通过rest请求获取一份服务清单（feth-registy=true时），并缓存30秒，之后再次向注册中心获取。

```yaml
eureka:
	client:
		registry-fetch-interval-seconds: 30
```

当Ribbon与eureka连用时，Ribbon的服务清单RibbonServiceList会被DiscoveryEnableNIWSServiceList重写，拓展成了Eureka注册中心获取服务列表。同时，NIWSDiscoveryPing来取代IPing，其职责为确认服务是否启动。

服务调用

服务在获取服务列表中包含了其实例名和该实例的元数据。默认Ribbon是通过轮询的方式进行调用，以达到负载均衡的效果。

服务下线

当eureka的客户端服务进行正常下线时，其会向注册中心发一个rest请求，告知注册中心服务下线，同时，注册中心也会通知其他服务该服务下线消息。

### 服务注册中心

失效剔除

如果是非正常服务停止的情况，注册中心会每60秒定时去清除清单中超时续约（90s）的服务。

自我保护

在注册中心运行期间，会统计心跳失败的比例在15分钟内是否低于85%，如果出现低于的情况，会将当前的实例注册信息保护起来，让这些实例不会过期。从而会导致一个问题就是客户端调用已经失效的服务，会出现错误，所以需要容错机制。

所以在本地开发的时候可以关闭保护机制，

```yaml
eureka:
	server:
		enable-self-preservation: false
```

### 随机数

```yaml
${random.int[10000,19999]}
${random.int(100)} #限制生成的数字小于10
${random.int[0,100]} #指定范围的数字
${random.value} #随机字符串
${random.long} #随机long
```

### 实例名配置

在使用服务高可用的时候，同一个服务的不同实例是根据主机名进行区分的，这样就会导致一台机子无法部署多台服务，解决这个问题的一种方式是通过更改服务的端口号，另一种是通过使用不同的实例名。

```yaml
eureka:
	instance:
		instanceId: ${spring.application.name}:${random.int}
```

### 健康检查

在eureka中的健康情况是根据服务提供者的心跳情况，只要续约正常，health一直是up状态。

使用需要spring-boot-actuator模块的/health端点。

```yaml
eureka:
	client:
		healthcheck:
			enabled: true
```

### 自定义Instance ID

在Spring Cloud中，服务的Instance ID的默认值是${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}} 。如果想要自定义这部分的内容，只需在微服务中配置eureka.instance.instance-id 属性即可，例如：

```yaml
spring:
  application:
    name: service-user
eureka:
  instance:
    instance-id: ${spring.cloud.client.ipAddress}:${server.port}	# 将Instance ID设置成IP:端口的形式
```

instanceID初始化代码

```
org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration
org.springframework.cloud.commons.util.IdUtils.getDefaultInstanceId(PropertyResolver)
org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean.getInstanceId()
```

### Tips

服务在启动时，会通过rest请求方式注册到eureka。注册信息是通过双层Map进行存储，第一层key为服务名，第二层的key为具体的实例名称。

other settings
```Yaml
eureka:
	instance:
		preferIpAddress: true #实例名称显示IP
```

```Yaml
eureka:
  client:
    healthcheck:
      enabled: true
```
[http://www.cnblogs.com/yangfei-beijing/p/6379258.html](http://www.cnblogs.com/yangfei-beijing/p/6379258.html)

### 源码学习

`EndpointUtils`

获取`serviceUrl`

```java
// 获取配置方式和DNS方式获取ServiceUrl地址
getDiscoveryServiceUrls();
// 遍历所有zone，获取所有ServiceUrl地址，将preferSameZone为true时，将instanceZone下的ServiceUrl放在最前面，也就是同一个与客户端同一个zone的ServiceUrl
getServiceUrlsFromConfig();
// 获取ServiceUrl地址，其实现是EurekaClientConfigBean类
getEurekaServerServiceUrls()；
```

`EurekaClientConfigBean`

该类包含了`eureka.client`的配置。包括一些定时任务配置。

```java
// 获取ServiceUrl地址
getEurekaServerServiceUrls()；
```

`com.netflix.discovery.DiscoveryClient`

最重要的类，定时任务和注册到注册中心等操作都是在这里完成的。

```java
// 这里启用服务获取，服务续约，服务注册等定时任务，同时InstanceInfoReplicator对象中run方法中的discoveryClient.register();进行了服务的注册
initScheduledTasks();
// 进行服务的获取，存在全量和非全量两种
fetchRegistry();
// 进行心跳的维持
HeartbeatThread.renew();
```

`EurekaInstanceConfigBean`

服务实例配置