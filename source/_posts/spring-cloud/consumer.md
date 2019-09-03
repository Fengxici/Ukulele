---
title: 第四章 服务消费者
date: 2019-08-31 09:21:32
tags: 
- spring cloud
- spring boot web
- ribbon
categories:
- spring cloud
- 教程
---

本章主要带大家动手制作一个微服务提供者项目，以备后期课程使用。

这里的提供者项目实际就是一个非常简单webmvc项目，在这个项目中我们将调用第三章创建的服务提供者项目。同样，我们也会将应用注册到到第二章创建的注册中心中去。

## 本章内容
> 客户端负载均衡：Ribbon

## 1 创建项目(porject)
此处命名为 **consumer-service**

## 2 修改pom依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>timing.springcloud</groupId>
    <artifactId>consumer-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>consumer-service</name>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
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
        <!-- 增加负载依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            <version>2.1.2.RELEASE</version>
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
package timing.springcloud.consumer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

## 4 Ribbon

将面向服务的REST模板请求自动转换成客户端负载均衡的服务调用。它是一个工具类框架，并不像Eureka那样可以独立部署。

通过Spring Cloud Ribbon我们在微服务架构中使用客户端负载均衡主要分为以下两步：

> 1 启动多个服务提供者服务实例并注册到一个注册中心或是多个相关的服务注册中心
> 2 服务消费者直接通过调用被@LoadBalanced注解修饰过的RestTemplate来实现面向服务的接口调用

下面让我们动手使用Ribbon来访问**第三章**创建的Get,Post,Put,Delete四个接口。

首先改造下启动类，开启负载均衡客户端
``` java
package timing.springcloud.consumer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
@SpringBootApplication
public class ConsumerApplication {
    @Bean
    //注册一个具有容错功能的RestTemplate
    @LoadBalanced
    //开启负载均衡客户端
    RestTemplate restTemplate() {
    return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```
接着添加controller包并新增ClientHelloController类
``` java
package timing.springcloud.consumer.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;
@RestController
public class ClientRibbonHelloController {
    private final RestTemplate restTemplate;
    @Autowired
    public ClientRibbonHelloController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    @GetMapping(value = "ribbon/get")
    public String ribbonGet() {
        return restTemplate.getForEntity("http://PROVIDER-SERVICE/get?para={0}", String.class, "client_ribbon").getBody();
    }
    @PostMapping(value = "ribbon/post")
    public String ribbonPost() {
        HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, Object> parammap = new LinkedMultiValueMap<>();
        parammap.add("para", "ribbon_client");
        HttpEntity<Map> entity = new HttpEntity<>(parammap,headers);
        return restTemplate.postForEntity("http://PROVIDER-SERVICE/post",entity, String.class ).getBody();
    }
    @PutMapping(value = "ribbon/put")
    public void ribbonPut() {
    HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, Object> parammap = new LinkedMultiValueMap<>();
        parammap.add("para", "ribbon_client");
        HttpEntity<Map> entity = new HttpEntity<>(parammap,headers);
        restTemplate.put("http://PROVIDER-SERVICE/put", entity);
    }
    @DeleteMapping(value = "ribbon/delete")
    public void ribbonDelete() {
        Map<String, String> params = new HashMap<>();
        params.put("para", "ribbon_client");
        restTemplate.delete("http://PROVIDER-SERVICE/delete/{para}", params);
    }
}
```
这里我们分别使用GET,POST,PUT,DELETE方法去访问**第三章**创建的提供者项目中对应请求类型的方法。

现在我们来验证一下。

首先启动一个注册中心（第二章创建的），简单起见，使用单机版。再启动**第三章**创建的服务提供者，将提供者注册至注册中心中。最后启动这里的服务消费者应用。全部启动之后，您的注册中心页面应该能够看到两个服务: **provider-sevice**和**consumer-service**

使用api测试工具，如**Restlet Client**或**postman**等，依次访问消费者的四个接口查看一下效果。

由于put和delete方法被我们定义为void无返回类型，而我们在提供者方法体内部做了控制台输出，所以我们需要到服务提供者的控制台查看。

本篇关于服务消费者就介绍到这里。后续章节我们会着重对其进行改造，希望大家多动手来实践。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。

