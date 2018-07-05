---
title: Asciidoctor Maven插件使用
tags: code
date: 2018-07-05
---

在项目应用中，我们会写很多文档去传递我们的设计思想、开发经验、采坑经历等等。使用Asciidoc的格式对非技术人员就不是那么的友好，或者说传递性、通用性与PDF和网页相比就差很多了。在JVM项目中可以使用Maven的插件方式将`.adoc`文件格式转化为PDF、HTML、EPUB等文件格式。

<!-- more -->

## 快速入门

**工程结构**

```txt
|doc-demo
|-src
|--main
|---asciidoc
|----.adoc文件
|---resources
|----images
|pom.xml
```

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.demo</groupId>
    <artifactId>docs</artifactId>
    <version>1.1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <asciidoctorj.version>1.5.6</asciidoctorj.version>
        <asciidoctorj.diagram.version>1.5.4.1</asciidoctorj.diagram.version>
        <jruby.version>1.7.26</jruby.version>
    </properties>
    <build>
        <!-- 默认命令，配置后可以直接使用mvn编译 -->
        <defaultGoal>process-resources</defaultGoal>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <targetPath>${project.build.directory}/book</targetPath>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>1.5.5</version>
                <executions>
                    <execution>
                        <id>output-html</id>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>html5</backend>
                            <sourceHighlighter>prettify</sourceHighlighter>
                            <attributes>
                                <toc>left</toc>
                                <icons>font</icons>
                                <sectanchors>true</sectanchors>
                                <!-- set the idprefix to blank -->
                                <idprefix/>
                            </attributes>
                        </configuration>
                    </execution>
                </executions>
                <dependencies>
                    <!-- Comment this section to use the default jruby artifact provided by the plugin -->
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                        <version>${jruby.version}</version>
                    </dependency>
                    <!-- Comment this section to use the default AsciidoctorJ artifact provided by the plugin -->
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj</artifactId>
                        <version>${asciidoctorj.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj-diagram</artifactId>
                        <version>${asciidoctorj.diagram.version}</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <outputDirectory>${project.build.directory}/book</outputDirectory>
                    <sourceDocumentName>book.adoc</sourceDocumentName>
                    <imagesDir>./</imagesDir>
                    <preserveDirectories>false</preserveDirectories>
                    <requires>
                        <require>asciidoctor-diagram</require>
                    </requires>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

**执行mvn命令**

```shell
mvn clean process-asciidoc
```

生成的HTML可以使用Http Server或者Nginx等服务进行部署，甚至可以使用Jenkins进行自动化部署。

## 生成PDF

**工程结构**

```txt
|doc-demo
|-src
|--main
|---asciidoc
|----data
|-----fonts
|-----themes
|----.adoc文件
|---resources
|----images
|pom.xml
```

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tairanchina.csp.dmp</groupId>
    <artifactId>docs</artifactId>
    <version>1.1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <asciidoctorj.version>1.5.6</asciidoctorj.version>
        <asciidoctorj.diagram.version>1.5.4.1</asciidoctorj.diagram.version>
        <jruby.version>1.7.26</jruby.version>
        <asciidoctorj.pdf.version>1.5.0-alpha-zh.16</asciidoctorj.pdf.version>
    </properties>
    <build>
        <!--https://github.com/asciidoctor/asciidoctor-maven-examples-->
        <!--https://github.com/asciidoctor/asciidoctor-maven-plugin/blob/master/README_zh-CN.adoc-->
        <!-- 默认命令，配置后可以直接使用mvn编译 -->
        <defaultGoal>process-resources</defaultGoal>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <targetPath>${project.build.directory}/book</targetPath>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>1.5.5</version>
                <executions>
                    <execution>
                        <id>output-html</id>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>html5</backend>
                            <sourceHighlighter>prettify</sourceHighlighter>
                            <attributes>
                                <toc>left</toc>
                                <icons>font</icons>
                                <sectanchors>true</sectanchors>
                                <!-- set the idprefix to blank -->
                                <idprefix/>
                            </attributes>
                        </configuration>
                    </execution>

                    <execution>
                        <id>output-pdf</id>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>pdf</backend>
                            <sourceHighlighter>coderay</sourceHighlighter>
                            <doctype>book</doctype>
                            <attributes>
                                <icons>font</icons>
                                <pagenums/>
                                <toc/>
                                <idprefix/>
                                <idseparator>-</idseparator>
                                <pdf-fontsdir>data/fonts</pdf-fontsdir>
                                <pdf-stylesdir>data/themes</pdf-stylesdir>
                                <pdf-style>cn</pdf-style>
                            </attributes>
                        </configuration>
                    </execution>
                </executions>
                <dependencies>
                    <!-- Comment this section to use the default jruby artifact provided by the plugin -->
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                        <version>${jruby.version}</version>
                    </dependency>
                    <!-- Comment this section to use the default AsciidoctorJ artifact provided by the plugin -->
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj</artifactId>
                        <version>${asciidoctorj.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj-diagram</artifactId>
                        <version>${asciidoctorj.diagram.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj-pdf</artifactId>
                        <version>${asciidoctorj.pdf.version}</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <outputDirectory>${project.build.directory}/book</outputDirectory>
                    <sourceDocumentName>book.adoc</sourceDocumentName>
                    <imagesDir>./</imagesDir>
                    <preserveDirectories>false</preserveDirectories>
                    <requires>
                        <require>asciidoctor-diagram</require>
                    </requires>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

**执行mvn命令**

```shell
mvn clean process-asciidoc
```

> 由于PDF格式插件没有安装中文字体，生成的PDF格式上会存在缺失，上方的fonts和themes可以对PDF的生成格式进行自定义，有时候为了方便，可以将其与`asciidoctorj-pdf`源码进行合并，手动打一个依赖包，放到自己的私服仓库中。

## 常见问题

- 在生成PDF的时候，可能code部分会存在很多空格的问题，一般产生这样的问题不是字体问题，而是编写格式有问题，可以选择将``符号去掉。

## 参考资料

[Example](https://github.com/asciidoctor/asciidoctor-maven-examples)

[Asciidoctor插件中文文档](https://github.com/asciidoctor/asciidoctor-maven-plugin/blob/master/README_zh-CN.adoc)

[Asciidoctor-PDF](https://github.com/asciidoctor/asciidoctor-pdf/blob/master/README.adoc)

[中文乱码问题解决方案](http://www.voidcn.com/article/p-qlpzlrqh-bkv.html)