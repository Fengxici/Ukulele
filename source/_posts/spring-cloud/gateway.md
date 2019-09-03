---
title: 第五章 网关
date: 2019-08-31 09:33:52
tags: 
- spring cloud
- spring boot web
- 网关 zuul 2
categories:
- spring cloud
- 教程
---
> API网关是一个服务器，是系统的唯一入口。从面向对象设计的角度看，它与外观模式类似。API网关封装了系统内部架构，为每个客户端提供一个定制的API。它可能还具有其它职责，如身份验证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。
> API网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也是提供REST/HTTP的访问API。服务端通过API-GW注册和管理服务。

由于Zuul2闭源，后又宣布开源。spring官方已经不再集成，自己做了一个spring-cloud-gateway项目，所以本章将分别使用Zuul2和spring-cloud-gateway来实现网关项目。

# 本章内容
> Zuul 2.0
> Spring Cloud Gateway

理论知识留待仔细研究，咱先还是先动手体验下。

首先我们创建一个父项目**gateway-demo** ，调整目录结构（删除src目录等）

修改下父项目的pom依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>timing.springcloud</groupId>
    <artifactId>gateway-demo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>gateway-demo</name>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
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
# 一 Zuul 2.0

## 1 创建模块(moudle) zuul-demo 
调整好目录结构,增加resources/applicaiton.yml等

## 2 修改pom依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>gateway-demo</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>zuul-demo</artifactId>
    <name>zuul-demo</name>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
    </dependencies>
</project>

```
## 3 添加启动类
``` java
package timing.springcloud.zuul;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}
```
没错，这里的**@EnableZuulProxy**即是实现网关的重点。
## 4 添加配置
resources/applicaiton.yml
``` yml
server:
    port: 10000
spring:
    application:
        name: gateway
eureka:
    instance:
        hostname: localhost
        prefer-ip-address: true
        instance-id: ${eureka.instance.hostname}:${server.port}
    client:
        serviceUrl:
            defaultZone: http://root:root@localhost:8080/eureka
#zuul配置，将所有的请求都转发至consumer-service
zuul:
    routes:
        provider:
            path: /**
            serviceId: consumer-service
```
## 5 查看效果
将第二章（注册中心）、第三章（服务提供者）、第四章（服务消费者）创建的项目依次运行起来。简单起见，使用单机版。全部运行起来之后，我们将这里的zuul-demo项目运行起来。全部运行起来，您的注册中心页面应该能够看到三个服务**provider-service**、**consumer-service**和**gateway**

我们点击gateway服务，访问get接口(http://localhost:10000/ribbon/get)

zuul将请求转发到了消费者服务consumer-service，消费者服务又通过ribbon调用了提供者provider-service的服务。这就是整个访问的流程。

好了，一个简单的网关服务就创建好了。后续的课程我们会对其进行进一步改造，欢迎继续关注。


# 二 spring-cloud-gateway

## 1 创建模块(module)cloud-gateway-demo 
调整项目结构，增加resources/application.yml

## 2 修改pom依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>gateway-demo</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cloud-gateway-demo</artifactId>
    <name>cloud-gateway-demo</name>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
    </dependencies>
</project>
```
## 3 添加启动类
``` java
package timing.springcloud.gateway;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
@SpringBootApplication
public class GatewayApplication {
    @Value("${test.uri:http://localhost:9090}")
    String uri;
    public static void main(String[] args){
        SpringApplication.run(GatewayApplication.class,args);
    }
}
```
## 4 添加路由规则
> 有两种方式：**代码** 和 **配置文件**

### 4.1 代码方式添加路由规则
改造上文的启动类
``` java
@SpringBootApplication
public class GatewayApplication {
    @Value("${test.uri:http://localhost:9090}")
    String uri;
    public static void main(String[] args){
        SpringApplication.run(GatewayApplication.class,args);
    }
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
        .route(t -> t.path("/baidu")
        .and()
        .uri("http://baidu.com"))
        .route(t -> t.path("/**")
        .and()
        .uri(uri))
        .build();
    }
}
```
注：配置文件中添加test.uri：
```yml
test:
    uri: http://127.0.0.1:9090
```

## 4.2 配置文件方式
``` yml
spring:
    application:
        name: gateway-service
    cloud:
        gateway:
            routes:
                - id: host_route
                  uri: http://www.baidu.com
                  predicates:
                    - Path=/baidu
                - id: host_route
                  uri: http://localhost:9090
                  predicates:
                    - Path=/**
```
以上两种方式都是配置两个路由，当请求以baidu开头则路由至baidu官网，其他所有请求路由至消费者服务。选择其一运行cloud-gateway-demo项目。
## 5 验证效果
> 1 访问 http://localhost:20000/ribbon/get 将会被路由至消费者服务，进而调用提供者服务得到**client_ribbon_get_service**输出
> 2 访问 http://localhost:20000/baidu 将会被路由至百度页面


本章关于路由的介绍都到此结束。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。