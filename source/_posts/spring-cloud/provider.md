---
title: 第三章 服务提供者
date: 2019-08-31 09:04:37
tags: 
- spring cloud
- spring boot web
categories:
- spring cloud
- 教程
---
本章主要带大家动手制作一个微服务提供者项目，以备后期课程使用。

这里的提供者项目实际就是一个非常简单webmvc项目，我们只实现controller层，并且将其注册到第二章创建的注册中心中去。

## 1 创建一个项目(project)
这里命名为 **provider-service**
## 2 修改pom依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>timimg.springcloud</groupId>
    <artifactId>provider-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>provider-service</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
            <version>2.0.3.RELEASE</version>
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
## 3 添加启动类
``` java
package timimg.springcloud.provider;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

## 4 创建HelloController
``` java
package timimg.springcloud.provider.controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletResponse;
@RestController
public class HelloController {
    @GetMapping("/get")
    @ResponseBody
    public String get(HttpServletResponse response, @RequestParam(value = "para") String para) {
        return para + "_get_service";
    }
    @PostMapping("/post")
    @ResponseBody
    public String post(HttpServletResponse response, @RequestParam(value = "para") String para) {
        return para + "_post_service";
    }
    @PutMapping("/put")
    @ResponseBody
    public String put(HttpServletResponse response, @RequestParam(value = "para") String para) {
        System.out.println(para + "_put_service");
        return para + "_put_service";
    }
    @DeleteMapping("/delete/{para}")
    @ResponseBody
    public String delete(HttpServletResponse response, @PathVariable(value = "para") String para) {
        System.out.println(para + "_delete_service");
        return para + "_delete_service";
    }
}
```
写了四个接口，对应增、删、改、查四个基本操作。并且！请注意，这里分别使用了GET、POST、PUT、DELETE四个方法。

## 5 REST与RESTful

REST全称是Representational State Transfer，中文意思是表述（编者注：通常译为表征）性状态转移。如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。

**REST是旧技术，新风格**

关于RESTful的更详细知识，希望大家自行去学习。这里主要谈一点，理解REST最重要的一点是：URL定位资源，用HTTP动词（GET,POST,DELETE,PUT）描述操作。

- 所有的新增操作对应的请求类型为POST
- 所有的修改操作对应的请求类型为PUT
- 所有的删除操作对应的请求类型为DELETE
- 所有的查询操作对应的请求类型为GET


spring cloud调用服务的方式为RESTful，所以请大家多少要了解下REST。

## 6 添加配置，将提供者注册至注册中心
resources/application.yml
``` yml
server:
    port: 8090
spring:
    application:
        name: provider-service
eureka:
    instance:
        hostname: localhost
        prefer-ip-address: true
        instance-id: ${eureka.instance.hostname}:${server.port}
    client:
        service-url:
            defaultZone: http://root:root@localhost:8080/eureka
```
首先我们改用**8090**端口。8080端口让给第二章中单机版的注册中心。

## 7 运行，查看效果

将第二章的注册中心配置为单机版，启动。完成后启动这里的提供者服务。读者可以使用postman分别对post,get,put,delete接口进行访问查看效果。

到此，一个简单的服务提供者制作好了。希望您比昨天又有了很大进步。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。

