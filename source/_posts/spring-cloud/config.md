---
title: 第十一章 配置中心
date: 2019-08-31 11:10:50
tags:
- spring cloud
- spring boot
- 配置中心
categories:
- spring cloud
- 教程
---
本章主要和大家一起动手体验一下Spring Cloud Config。通过本章的学习您将获得
> Spring Cloud Config服务端搭建

> Spring Cloud Config客户端搭建

> Spring Cloud Config实时刷新配置

在微服务体系中，单个服务可以集群部署，随着集群数量的增大，配置变更时每个服务实例都需要修改，十分麻烦，并且也容易出错。配置中心便是解决这类问题的灵丹妙药。

Spring Cloud Config是Spring官方提供的用来为分布式系统中的基础设施和微服务应用提供集中化配置支持的组件，它也是分为服务端和客户端两个部分。服务端即是我们所说的配置中心，它是一个独立的微服务应用，可以单独部署，它的主要功能是来连接配置仓库并为客户端提供获取配置信息、加解密信息等访问接口；客户端嵌入在微服务架构中的各个微服务应用或基础设施中，他们通过指定的配置中心来管理应用资源与业务相关配置内容，并在启动的时候从配置中获取和加载配置信息。

下面我们来动手体验一下配置中心的搭建过程。

---

## 1 创建父项目config-demo;

## 2 创建服务端module config-server;

## 3 引入依赖
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>2.0.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
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
```
这里的spring-cloud-config-server即是配置中心服务端的关键依赖。

## 4 创建启动类
``` java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

注意，**@EnableConfigServer**注解即是开启配置服务的关键代码。

## 5 添加配置文件
配置中心的配置仓库可以支持多种方式：git、svn、本地文件系统。这里为了简单起见，我们先使用本地文件系统方式。
```yml
server:
    port: 8888
spring:
    application:
        name: config-local-service
    profiles:
        active: native
    cloud:
        config:
            server:
                native:
                    searchLocations: classpath:/config
```                    

在resources目录下创建一个config文件夹

我们先放个配置文件看看效果，由于后面会使用客户端检验效果，这里先创建一个名为client-config-dev.yml文件，配置内容如下：
``` yml
client:
    hello: hello i am dev update.
```
运行server-demo应用，浏览器打开localhost:8888/client-config/dev 可以看到 包含以上配置的json字符串输出，说明配置文件文件已经能够正常读取了。接下来我们创建一个客户端来获取配置值。

## 6 创建客户端config-client

## 7 引入依赖
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.0.0.RELEASE</version>
    </dependency>
    <!-- 从配置服务器读取配置 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.0.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
## 8 添加启动类
``` java
@SpringBootApplication
public class ConfigClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }
}
```
## 9 创建一个controller，通过浏览器来显示读取到的配置
``` java
@RestController
class HelloController {
    @Value("${client.hello}")
    private String hello;
    @RequestMapping("/hello")
    public String from() {
        return this.hello;
    }
}
```
## 10 添加配置文件
> application.yml
``` yml
spring:
    application:
        name: config-client
server:
    port: 8000
```
> bootstrap.yml
```yml
spring:
    cloud:
        config:
            name: client-config
        profile: dev
        uri: http://localhost:8888/
```

将客户端程序命名为client-config。之前server端创建的client-config-dev.yml名称即和这里的名称有关，dev表示的是开发环境，配置中心也支持多环境配置。

## 11 运行客户端程序
浏览器访问localhost:8000/hello,将会看到**hello i am dev update**,成功读取到了服务端的配置。

到这里配置中心的客户端和服务端就搭建完成了。

但是，我们试想一下，如果服务端的配置发生了变化，客户端会发生变化么？

至少目前服务端配置变化，客户端是不会发生变化的。有没有解决的方法呢？

这时Spring Cloud Bus就派上用场了。通过Spring Cloud Bus可以实现配置内容的实时更新。下面我们来改造项目，体验下效果。

## 12 引入依赖
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```

**server和client项目都要引入**

## 13 添加配置
连接到消息队列服务器，这里我们选择的是RabbitMQ。同时我们启动actuator的刷新端点。
```yml
Spring:
    rabbitmq:
        host: 127.0.0.1
        port: 5672
        username: guest
        password: guest
    cloud:
        bus:
            trace:
                enabled: true
        enabled: true
management:
    endpoints:
        web:
            exposure:
                include: "*"
```
## 14 修改HelloController类
``` java
@RestController
@RefreshScope
class HelloController {
    @Value("${client.hello}")
    private String hello;
    @RequestMapping("/hello")
    public String from() {
        return this.hello;
    }
}
```
在类的前面加上**@RefreshScope**注解。

## 15 查看效果
一次启动server和client两个应用，访问client的hello接口，看到的内容应该是

**hello i am dev update.**

修改服务端target目录下的client-config-dev.yml文件，然后使用Restlet Client,postman等工具访问下配置中心的刷新端点localhost:8000/actuator/bus-refresh。

再次刷新hello接口，您将看到的结果为

**hello i am dev update. now i am updated.**

好了，服务端的配置更新，客户端自动进行了刷新。

到此，配置中心就介绍完了。

如前文所述，配置仓库可以是svn，git，本地文件等多种方式，这里只介绍了本地文件一种，其他方式大家可以自行去尝试，并且也可以将配置中心注册至注册中心中去。这里就不一一介绍，感兴趣的读者可以自行翻阅资料去完成。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。