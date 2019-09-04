---
title: 第十章 服务监控
date: 2019-08-31 10:56:43
tags:
- spring cloud
- spring boot
- spring boot admin
categories:
- spring cloud
- 教程
---
由于spring cloud微服务架构都是基于spring boot构建的，所以我们就可以是很方便使用一种统一的方式来监控这些服务。Spring Boot Admin正式这样一个用于管理和监控一个或多个Spring Boot程序的组件。其在 Spring Boot Actuator 的基础上提供了简洁的可视化 WEB UI，提供如下功能：
> 显示 name/id 和版本号
> 显示在线状态
> Logging 日志级别管理
> JMX beans 管理
> Threads 会话和线程管理
> Trace 应用请求跟踪
> 应用运行参数信息，如：
> > Java 系统属性
> > Java 环境变量属性
> > 内存信息
> > Spring 环境属性

---

Spring Boot Admin分为服务端和客户端两部分。接下来我们依次创建这两部分。

## 1 创建父项目monitor-demo
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>timing.springcloud</groupId>
  <artifactId>monitor-demo</artifactId>
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>monitor-demo</name>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.7.RELEASE</version>
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
## 2 创建server端module server-demo

## 3 server-demo的依赖如下
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>monitor-demo</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>server-demo</artifactId>
    <name>server-demo</name>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
            <version>2.1.6</version>
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
</project>
```
## 4 创建启动类
``` java
@SpringBootApplication
@EnableAdminServer
public class ServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServerApplication.class, args);
    }
}
```
**@EnableAdminServer**是这里的关键
## 5 添加配置
``` yml
server:
    port: 8000
spring:
    application:
        name: server-demo
eureka:
    instance:
        hostname: localhost
        prefer-ip-address: true
        instance-id: ${eureka.instance.hostname}:${server.port}
    client:
        serviceUrl:
            defaultZone: http://root:root@localhost:8080/eureka
management:
    endpoints:
        web:
            exposure:
                include: "*"
# SpringBootActuator监控暴露所有接口
    endpoint:
        health:
            show-details: ALWAYS
```
至此**服务端**创建完毕。接下来创建**客户端**。

## 6 创建client-demo module，pom依赖如下
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>monitor-demo</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>client-demo</artifactId>
    <name>client-demo</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
</project>
```
## 7 添加启动类
``` java
@SpringBootApplication
public class ClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }
}
```
## 8 添加配置
```yml
server:
    port: 7000
spring:
    application:
        name: client-demo
eureka:
    instance:
        hostname: localhost
        prefer-ip-address: true
        instance-id: ${eureka.instance.hostname}:${server.port}
client:
    service-url:
    defaultZone: http://root:root@localhost:8080/eureka
management:
    endpoints:
        web:
            exposure:
                include: "*"
# SpringBootActuator监控暴露所有接口
    endpoint:
        health:
            show-details: ALWAYS
```

诚如前文，Spring Boot Admin是在Spring Boot Actuator 的基础上提供的简洁的可视化 WEB UI，所以我们将**服务端和客户端的actuator端点全部打开**了。

## 9 查看效果

> 启动一个注册中心（第二章）

> 启动服务端

> 启动客户端

> 浏览器输入localhost:8000，您将看到服务列表界面

点击wallboard将会看到两个巨大的绿色六边形。至此客户端client-demo和服务端server-demo服务已在监控之列。

选择其中一个服务如server-demo，点击。您将可以非常直观的看出服务的各项指标。
> 事实上，Spring Boot Admin 可以自动检测注册中心下的所有服务，所以我们的客户端并不需要加入spring-boot-admin-starter-client依赖

本章关于Spring Boot Admin 的演示就到这里。下一章我们将介绍微服务一个非常重要的组件——配置中心。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。