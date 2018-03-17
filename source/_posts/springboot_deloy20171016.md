---
title: Spring Boot启动方式与部署
tags: spring-boot
date: 2017-10-16
---

Spring Boot为我们提供很多便捷的启动和配置方式。本文就来好好说一下这两方面。

<!-- more -->

### 启动方式

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class GirlApplication {
	public static void main(String[] args) {
		SpringApplication.run(GirlApplication.class, args);
	}
}
```
方法一：直接在程序中run

方法二：在命令行中切换到工程所在目录，`mvn spring-boot:run`

方法三：先`mvn install`编译工程，之后切换到target路径下，使用` java -jar jar包名`

```shell
java -jar target/first-project-1.0.0.jar --spring.profile.active=prod
```

> 在Ctrl+C之后，服务即停止

方法四：

```shell
nohup -Dspring.profiles.active=dev -jar XXX.jar >/dev/null &
# 指定log
java -jar spring-boot01-1.0-SNAPSHOT.jar > log.file 2>&1 &
# 配置服务内存信息
java -server -Xms8000m -Xmx8000m -jar luckydrawall-0.1.1.jar --spring.profiles.active=rel-Xmx8000m -jar luckydrawall-0.1.1.jar --spring.profiles.active=rel
```

方法五：外部Tomcat部署（不推荐）

1、将项目的启动类Application.java继承SpringBootServletInitializer并重写configure方法

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}
```
2、在pom.xml文件中，project下面增加package标签
```xml
<packaging>war</packaging>
```
3、还是在pom.xml文件中，dependencies下面添加
```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>complied</scope>
</dependency>
```
这样，只需要以上3步就可以打包成war包，并且部署到tomcat中了。需要注意的是这样部署的request url需要在端口后加上项目的名字才能正常访问。spring-boot更加强大的一点就是：即便项目是以上配置，依然可以用内嵌的tomcat来调试，启动命令和以前没变，还是：`mvn spring-boot:run`。
如果需要在springboot中加上request前缀，需要在application.properties中添加`server.contextPath=/prefix/`即可。其中prefix为前缀名。这个前缀会在war包中失效，取而代之的是war包名称，如果war包名称和prefix相同的话，那么调试环境和正式部署环境就是一个request地址了。

### 部署

由于Spring Boot内置了Tomcat，从而可以直接使用jar的方式进行部署。启动命令在上方进行了说明。部署这一环节重要就是**配置文件**。

Spring Boot 所提供的配置优先级顺序比较复杂。按照优先级**从高到低**的顺序，具体的列表如下所示。

1. 命令行参数。
2. 通过 System.getProperties() 获取的 Java 系统参数。
3. 操作系统环境变量。
4. 从 java:comp/env 得到的 JNDI 属性。
5. 通过 RandomValuePropertySource 生成的“random.*”属性。
6. 应用jar 文件之外的属性文件。(通过spring.config.location参数)
7. 应用jar 文件内部的属性文件。
8. 在应用配置 Java 类（包含“@Configuration”注解的 Java 类）中通过“@PropertySource”注解声明的属性文件。
9. 通过“SpringApplication.setDefaultProperties”声明的默认属性。

说明：

1）Spring Boot应用在启动命令中使用`--`开头的命令行参数，可修改应用的配置。

```shell
java -server -Xms8000m -Xmx8000m -jar luckydrawall-0.1.1.jar --spring.profiles.active=rel-Xmx8000m 
```

使用如下代码进行关闭

```java
SpringApplication.setAddCommandLineProperties(false)
```

6）属性文件是比较推荐的配置方式。Spring Boot在启动时会对如下目录进行搜查，读取相应配置文件。**优先级从高到低**。

- 当前jar目录的“/config”子目录
- 当前jar目录
- classpath 中的“/config”包
- classpath

可以通过“spring.config.name”配置属性来指定不同的属性文件名称。也可以通过“**spring.config.location**”来添加**额外的属性文件的搜索路径**。如果应用中包含多个 profile，可以为每个 profile 定义各自的属性文件，按照“application-{profile}”来命名。

```shell
java -jar demo.jar --spring.config.location=/path/test_evn.properties
```

#### 使用**Profile**区分环境

在Spring Boot中可以使用application.yml，application-default.yml，application-dev.yml，application-test.yml进行不同环境的配置。默认时，会读取application.yml，application-default.yml这两个文件中的配置，优先级高的会覆盖优先级低的配置。无论切换到哪个环境，指定的环境的配置的优先级是最高的。

可以使用`spring.profiles.active=dev`指定环境。

### 推荐阅读：

[Spring Boot 配置优先级顺序](http://www.cnblogs.com/softidea/p/5759180.html)

