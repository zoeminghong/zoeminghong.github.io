---
title: Feign 如何支持进行文件上传
tags: spring-cloud
date: 2017-12-01
---

最近，别的项目组提出需要`SDK`，就利用Feign做了一个，在此期间发现上传文件是一个坑，正常的实现是无法支持文件上传，需要进行对资源有一个`Convert`。为了避免大家像我一样，继续掉坑里，就出现了这篇文章的初衷。

<!-- more -->

### 入门

- 在SDK工程处，添加包依赖

```xml
        <dependency>
            <groupId>io.github.openfeign.form</groupId>
            <artifactId>feign-form</artifactId>
            <version>3.0.1</version>
        </dependency>
        <dependency>
            <groupId>io.github.openfeign.form</groupId>
            <artifactId>feign-form-spring</artifactId>
            <version>3.0.1</version>
        </dependency>
```

- 在SDK工程处，创建一个Configuration

```java
import feign.codec.Encoder;
import feign.form.spring.SpringFormEncoder;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.cloud.netflix.feign.support.SpringEncoder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MultipartSupportConfig {

    @Autowired
    private ObjectFactory<HttpMessageConverters> messageConverters;

    @Bean
    public Encoder feignFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(messageConverters));
    }

}
```

期初在网上看到是使用下方的注入方式，一直不成功，在走头部路下，尝试了👆者方案成功了。

```java
@FeignClient(name = "demo",configuration=MultipartSupportConfig.class)
public interface SignBaseCommonClient {
  
}
```

- 修改接口

```java
@FeignClient(name = "demo")
public interface FeginExample {    
@PostMapping(value = "images", consumes = MULTIPART_FORM_DATA_VALUE)
 Resp<String> uploadImage(
            @RequestParam MultipartFile image,
            @RequestParam("id") String id);
}
```

`@RequestPart`与`@RequestParam`效果是一样的，大家就不用花时间在这上面了。

- 修改服务器接口

```java
@RestController
public class FeginServiceExample {
  @PostMapping(value = "images", consumes = MULTIPART_FORM_DATA_VALUE)
    public Resp<String> uploadImage(
            @RequestParam("image") MultipartFile image,
            @RequestParam("id") String id,
            HttpServletRequest request) {
              return Resp.success(null);
            }
}
```

- 在启动类添加`@EnableFeignClients`

这个就不用多说了吧，😆

### 常见问题：

- HTTP Status 400 - Required request part 'file' is not present

> 请求文件参数的名称与实际接口接受名称不一致

- feign.codec.EncodeException: Could not write request: no suitable HttpMessageConverter found for request type [org.springframework.mock.web.MockMultipartFile] and content type [multipart/form-data]

> 转换器没有生效，检查一下`MultipartSupportConfig`

