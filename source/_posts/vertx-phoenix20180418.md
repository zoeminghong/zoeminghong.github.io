---
title: Vertx-Phoenix实践
tags: code
date: 2018-04-18
---

## 前言

### Vertx

Vertx是一个高效的异步框架，支持Java、Scala、JavaScript、Kotlin等多种语言。在非性能调优的场景下，TPS可以高达2-3万，同时，支持多种数据源也提供了异步支持。

<!-- more -->

### Phoenix

大数据的同学肯定对其很了解，是Apache基金会下的顶级工程，Phoenix帮助Hbase提供了SQL语法的支持，让难用的Hbase变得简单易用。

### 场景出发点

#### 目标

在项目应用中，为了达到简单、高效的接口化查询功能。

#### 现状

- 使用HBase作为数据的持久化
- 场景对接口的TPS要求比较高
- 操作方式简单

#### 问题与方案

- Hbase是一种很好的大数据存储方案，但是其不支持SQL化操作，在开源解决方案中提供了Phoenix方案，文档和社区都比较活跃，故优先采用了
- 需要接口化和高TPS，使用单纯的Spring Boot无法实现目标，Vertx之前就在项目中使用，对其性能有所了解，同时支持Web应用，可以Spring Boot一起使用，故而选之

## Vertx-Phoenix实现

>  只对涉及Phoenix方面进行讲解，通过Scala进行编写

### 依赖Pom

```xml
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-mysql-postgresql-client</artifactId>
</dependency>
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-jdbc-client</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.13.1-HBase-1.3</version>
</dependency>
```

### 实现

```scala
class HbaseDatabase(vertx: Vertx, dbConfig: DBInfoEntity, queryTimeout: Long)
  extends AbstractDatabase(vertx, dbConfig, queryTimeout) with LazyLogging {

  var hbaseClient: JDBCClient = _
  init(vertx, dbConfig, queryTimeout)

  override def init(vertx: Vertx, dbConfig: DBInfoEntity, queryTimeout: Long): Unit = {
    val HbaseClientConfig: JsonObject = new JsonObject()
      .put("url", dbConfig.getHost)
      .put("user", "")
      .put("password", "")
      .put("max_idle_time", queryTimeout)
      .put("driver_class", "org.apache.phoenix.jdbc.PhoenixDriver")
    this.hbaseClient = JDBCClient.createShared(vertx, HbaseClientConfig)
  }

  override def action(bodyInformation: BodyInformationVO, callback: Resp[Object] => Unit): Unit = {
    hbaseClient.getConnection(new Handler[AsyncResult[SQLConnection]]() {
      override def handle(res: AsyncResult[SQLConnection]): Unit = {
        if (res.succeeded()) {
          val connection = res.result()
          connection.query(bodyInformation.command, new Handler[AsyncResult[ResultSet]]() {
            override def handle(result: AsyncResult[ResultSet]): Unit = {
              if (result.succeeded()) {
                setMetaData(result.result().getColumnNames, CacheContainer.getMetadatas)
                callback(Resp.success(result.result().getRows()))
              } else {
                logger.error("Get HBase value is error", result.cause())
                callback("Get HBase value is error")
              }
            }
          })
        } else {
          logger.error("Get HBase connect is error", res.cause())
          callback("Get HBase connect is error")
        }
      }
    })
  }
```

### 相关配置说明

#### URL格式

```Txt
jdbc:phoenix:host1,host2:2181:/hbase
```

