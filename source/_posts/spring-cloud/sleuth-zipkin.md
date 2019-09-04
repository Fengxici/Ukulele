---
title: 第九章 链路跟踪
date: 2019-08-31 10:43:17
tags:
- spring cloud
- spring boot
- sleuth
- zipkin
categories:
- spring cloud
- 教程
---

随着业务的发展，微服务的系统规模会变得越来越大，各个微服务之间的调用关系也会随之变得越来越错综复杂。通常，一个由客户端发起的请求在后端系统中会经过多个不同的微服务调用来协同产生最后的请求结果，在复杂的微服务架构系统中，几乎每一个前端请求都会形成一条复杂的分布式服务调用链路，在每条链路中任何一个依赖服务出现延迟过高或错误的时候都有可能引起请求最后的失败，这个时候，对于每个请求，全链路调用的追踪就变得越来越重要，通过实现对请求调用的跟踪可以很好的帮助我们快速发现错误根源以及监控分析每条请求链路上的性能瓶颈等。

本章将介绍Sleuth和Zipkin，二者整合一起使用，可以直观的搜索跟踪信息和分析请求链路明细。

下面就让我们动手体验。

---

在这之前我们先要做些准备中作。

> 1 编排好三个服务，服务提供这provider-service，使用7000端口；服务消费者customer-service，使用8000端口；链路追踪服务trace-service，使用9000端口。它们都向使用8080端口的Eureka注册中心服务注册。

> 2 provider-demo与第三章内容相同。

> 3 customer-demo与第四章内容相同。

现在让我们正式开始。

## 1 创建父项目trace-demo，修改pom如下：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>timing.springcloud</groupId>
  <artifactId>trace-demo</artifactId>
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>trace-demo</name>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.7.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
  <profiles>
    <profile>
      <id>spring</id>
      <repositories>
        <repository>
          <id>spring-snapshots</id>
          <name>Spring Snapshots</name>
          <url>https://repo.spring.io/libs-snapshot-local</url>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
          <releases>
            <enabled>false</enabled>
          </releases>
        </repository>
        <repository>
          <id>spring-milestones</id>
          <name>Spring Milestones</name>
          <url>https://repo.spring.io/libs-milestone-local</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
        <repository>
          <id>spring-releases</id>
          <name>Spring Releases</name>
          <url>https://repo.spring.io/release</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>spring-snapshots</id>
          <name>Spring Snapshots</name>
          <url>https://repo.spring.io/libs-snapshot-local</url>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
          <releases>
            <enabled>false</enabled>
          </releases>
        </pluginRepository>
        <pluginRepository>
          <id>spring-milestones</id>
          <name>Spring Milestones</name>
          <url>https://repo.spring.io/libs-milestone-local</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </pluginRepository>
        <pluginRepository>
          <id>spring-releases</id>
          <name>Spring Releases</name>
          <url>https://repo.spring.io/libs-release-local</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
    <profile>
      <id>java9+</id>
      <activation>
        <jdk>[9,)</jdk>
      </activation>
      <dependencies>
        <dependency>
          <groupId>javax.activation</groupId>
          <artifactId>javax.activation-api</artifactId>
        </dependency>
        <dependency>
          <groupId>javax.xml.bind</groupId>
          <artifactId>jaxb-api</artifactId>
        </dependency>
        <dependency>
          <groupId>org.glassfish.jaxb</groupId>
          <artifactId>jaxb-runtime</artifactId>
          <optional>true</optional>
        </dependency>
      </dependencies>
    </profile>
  </profiles>
</project>
```
## 2 创建provider-demo module，内容与第三章相同

## 3 创建customer-demo module，内容与第四章相同。

## 4 创建sleuth-zipkin-demo module。

## 5 修改sleuth-zipkin-demo的pom依赖
``` xml
……
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        <version>2.0.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-server</artifactId>
        <version>2.9.4</version>
    </dependency>
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-autoconfigure-ui</artifactId>
        <version>2.9.4</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
……
```
## 6 添加sleuth-zipkin-demo的启动类。
``` java
package timing.springcloud.sleuth.zipkin;

import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import zipkin2.server.internal.EnableZipkinServer;
import zipkin2.server.internal.RegisterZipkinHealthIndicators;

@SpringBootApplication
@EnableZipkinServer
public class TraceApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(TraceApplication.class)
                .listeners(new RegisterZipkinHealthIndicators())
                .properties("spring.config.name=zipkin-server")
                .run(args);
    }
}

```
## 7 添加sleuth-zipkin-demo的配置
注意：必须是**resources/bootstratp.yml**文件，而不是applicaition.yml文件
``` yml
QUERY_PORT: 9000
eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${QUERY_PORT:9411}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8080/eureka

management:
  metrics:
    web:
      server:
        auto-time-requests: false

spring:
  application:
    name: trace-server
  profiles:
    include: shared
```
追踪服务建好之后，我们需要改造provider-service和customer-service：引入sleuth和zipkin依赖，并且增加配置以告知其Zipkin Server位置。

## 8 分别在provider-demo和customer-demo的pom中增加sleuth和zipkin依赖
``` xml
<!--添加链路追踪支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
```
此依赖即包含了zipkin和sleuth

## 9 分别在provider-demo和customer-demo的配置文件里添加Zipkin Server配置 
``` yml
spring:
    zipkin:
        base-url: http://localhost:9000
    sleuth:
        sampler:
            probability: 1.0
```

至此，全部项目创建完毕。

---

我们来查看下效果。

> 1 先运行起来一个Eureka注册中心，使用8080端口。

> 2 运行起trace-demo，

> 3 运行起provider-service

> 4 运行起customer-service

> 5 浏览器打开localhost:9000/zipkin/index.html，点击“查找调用链”

## 6 多次访问provide-service和customer-service的接口

整个调用信息被非常直观的罗列出来了。

本章关于链路追踪的实现就介绍到这里。

值得注意的是：

**`从spring Cloud为F版本开始，已经不需要自己构建Zipkin Server了，只需要下载jar运行即可。`** 详情请参看官网[Zipkin](https://zipkin.io/pages/quickstart.html)

个人能力有限，才疏学浅，如有疏漏，烦请斧正。