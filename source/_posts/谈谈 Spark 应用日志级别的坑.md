---
title: 谈谈 Spark 应用日志级别的坑
tags: code
date: 2019-06-24
---

> 环境说明：HDP 3.0 + Kerberos + Livy

根据 [Spark](http://spark.apache.org/docs/latest/running-on-yarn.html#debugging-your-application) 官方文档的指引，清楚的知道存在三种方式可以对应用的日志级别进行调整。

<!-- more -->

- upload a custom `log4j.properties` using `spark-submit`, by adding it to the `--files` list of files to be uploaded with the application.
- add `-Dlog4j.configuration=<location of configuration file>` to `spark.driver.extraJavaOptions` (for the driver) or `spark.executor.extraJavaOptions` (for executors). Note that if using a file, the `file:` protocol should be explicitly provided, and the file needs to exist locally on all the nodes.
- update the `$SPARK_CONF_DIR/log4j.properties` file and it will be automatically uploaded along with the other configurations. Note that other 2 options has higher priority than this option if multiple options are specified.

第三种方案是我们最不希望看到的选择，因而选择了第一、二两种进行尝试。

## 第一种方案

**过程：**

1. 将编辑准备好的 `log4j-error.properties` 文件上传到 HDFS
2. 授予接下来启动 Spark 应用的用户读取权限
3. 启动参数中，添加 `"files":["/logfile/log4j-error.properties"]`  参数，来指定 `log4j.properties` 文件路径。
4. 启动参数中，添加 `"spark.driver.extraJavaOptions": "-Dlog4j.configuration=log4j-error.properties","spark.executor.extraJavaOptions": "-Dlog4j.configuration=log4j-error.properties"` ，相对路径即可
5. 使用 [Livy](https://livy.apache.org/docs/latest/rest-api.html) 进行启动应用

**这里的可能遇到的问题：**

- `-files` 参数是数组，不是简单的 String
- `spark.driver.extraJavaOptions` 用于 Driver 的日志级别文件的指定，`"spark.executor.extraJavaOptions` 用于 Executor 的日志级别文件的指定。可以单独分别指定，支持相对路径。
- HDFS 下文件的权限一定要注意

## 第二种方案

**过程：**

1. 将编辑准备好的 `log4j-error.properties` 文件上传到 `Spark Server` 所在服务器的 `$SPARK_CONF_DIR/` 目录下
2. 授予  `log4j-error.properties` 文件读取权限，粗暴一点直接设置为 777
3. 启动参数中，添加 `"spark.driver.extraJavaOptions": "-Dlog4j.configuration=log4j-error.properties","spark.executor.extraJavaOptions": "-Dlog4j.configuration=log4j-error.properties"` ，如果相对路径不生效，可以使用绝对路径( `file:/spark/conf/log4j-error.properties` )
4. 使用 [Livy](https://livy.apache.org/docs/latest/rest-api.html) 进行启动应用

**这里的可能遇到的问题：**

- 读取的权限，一定要启动应用的用户拥有权限
- 所有 Spark 节点下都要有日志文件哦

## 第三种方案

**过程：**

1. 在 `HDP Ambari `页面 Spark 下，对日志配置进行相应的修改
2. 重启 Spark 服务使其生效

**这里的可能遇到的问题：**

- 优先级该方案是三者中最低的，前两种都可以覆盖该种方案

## 谈谈 `log4j` 配置

### 自定义某路径下的日志级别

比如：希望 `com.zerostech.demo` 路径下日志级别为 ERROR

```properties
log4j.logger.com.zerostech.demo=ERROR
```

