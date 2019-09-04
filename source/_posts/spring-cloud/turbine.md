---
title: 第八章 服务熔断汇总
date: 2019-08-31 10:31:13
tags: 
- spring cloud
- spring boot
- hystrix
- turbine
categories:
- spring cloud
- 教程
---

> 汇总熔断实时监控数据

本章我们将使用Hystix的集群工具——Turbine创建一个新项目用来收集Hystrix的监控数据。

端起小键盘，跟我一起愉快的开始敲代码吧。

---

## 1 创建消费者项目，命名为turbine-demo，并调整好项目结构。

## 2 修改pom，引入依赖。
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>timing.springcloud</groupId>
    <artifactId>turbine-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>turbine-demo</name>

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
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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
毫无疑问，**spring-cloud-starter-netflix-turbine**即是核心依赖。

## 3 创建启动类，并开启Turbine。
``` java
@SpringBootApplication
@EnableTurbine
@EnableHystrixDashboard
public class TurbineApplication {
    public static void main(String[] args) {
        SpringApplication.run(TurbineApplication.class, args);
    }
}
```
基于Hystrix dashboard，所以这里也要加上**@EnableHystrixDashboard**注解。

## 4 添加配置
``` yml
server:
    port: 9080
spring:
    application:
        name: turbine-dashboard
eureka:
    instance:
        hostname: localhost
        prefer-ip-address: true
        instance-id: ${eureka.instance.hostname}:${server.port}
    client:
        service-url:
            defaultZone: http://root:root@localhost:8080/eureka
turbine:
    appConfig: hystrix-dashboard-service
    aggregator:
        clusterConfig: default
        clusterNameExpression: new String("default")
    instanceUrlSuffix: /hystrix.stream
```
turbine配置节就是关于turbine的配置。这里需要说明的是turbine.appConfig的值是你要监控的服务，这里我们是要监控第七章的hystrix-dashboard-service，如果是多个服务，使用逗号隔开。

---

好了，项目创建完毕，运行起来查看下效果吧。

> 1 启动注册中心（第二章）
> 2 启动提供者服务（第三章）
> 3 启动至少两个hystrix-dashboard-service（第七章）
> 4 启动本章项目。
> 5 在浏览器打开localhost:9090/ribbon/get 备用
> 6 浏览器打开localhost:9080/hystrix 可以看大这里跟第七章时一样。
> 7 在最长的输入框中输入localhost:9080/turbine.stream，点击Monitor Stream，此时你将看到跟第七章类似的界面，还是那只可爱的小熊.

仔细观察可以看到这里可以看到hystrix-dashboard-service示例启动了两个。

刷新hystrix-dashboard-service接口观察效果，也可以修改两个或这多hystrix-dashboard-service的spring.application.name配置观察下Turbine界面的不同。


本章关于Turbine的介绍就到这里。有了调用成功与否的监控，我们的微服务框架也已经具备了一定的完整性了，接下来我们将继续完善，下一章我们将介绍一个请求从gateway进入后，服务间完整的调用过程跟踪。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。
