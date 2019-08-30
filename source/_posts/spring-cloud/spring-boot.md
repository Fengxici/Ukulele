---
title: 第一章 spring boot 简介
date: 2019-08-29 10:04:47
tags: 
- spring cloud
- spring boot 2
categories:
- spring cloud
- 教程
---

> BUILD ANYTHING

官网指导书：
https://docs.spring.io/spring-boot/docs/current/reference/html/

本篇及后续文章均基于Spring Boot 2系列，简称Spring Boot，如非特别指定，均代指Spring Boot 2.
# 目标

> 1 Spring Boot 2 项目创建、打包、运行
> 2 Spring Boot 2 日志配置
> 3 Spring Boot 2 热部署
> 4 Spring Boot 2 运维与监控

# 一、如何创建Spring Boot 2 项目
Spring Boot项目的创建有多种方式，这里主要介绍**四种**
## 1.官方站点：http://start.spring.io
按照页面指引，依次选择

> - 项目类型: maven或者gradle。 使用自己熟悉的构建环境，本篇选择maven
> - 开发语言：Java、Kotlin或Groovy。使用自己熟悉的开发语言，本篇选择Java
> - Spring Boot 版本：选择最新的发布版本。（学习目的可选择最新版本）
> - maven相关配置：groupId,artifectId,version等
> - 其他依赖：web,jpa等，按需选择

## 2.Spring Tool Suite (STS)
>该工具为定制版的Eclipse，下载地址：http://spring.io/tools/sts/all
创建界面的内容与上文相似

## 3.各集成开发工具引导界面，如(IDEA,Eclipse)
> new project，选择Spring Initializr,后续创建界面与上文相似。

## 4.手工定制
> 在各集成开发工具中使用Maven或Gradle的方式手工创建空项目

本章主要以此方式来介绍Spring Boot 2 项目创建，请继续往下查看具体过程。

# 二、动手实践(idea,maven)
## 1.创建项目
> 1 打开 New project
> 2 左侧选择maven
> 3 右侧先选择java版本，**必须1.8以上**。
> 4 再勾选**Create from archetype**.
> 5 选择**org.apache.maven.archetypes:maven-archetype-quickstart**
> 6 点击Next
> 7 以此填写GroupId,ArtifectId,Version(可默认)
> 8 点击Next
> 9 依次选择自己Maven的安装位置(加过环境变量的会有默认值),用户的setting文件和本地仓库地址（如果不能选择，试试勾选Override)
> 10 点击Next
> 11 依次填写项目名称和保存位置
> 12 点击Finish
> 13 稍等片刻项目即可创建完毕。

项目创建完成后，可能main目录下没有资源目录resources，如果没有，需要手工创建，并将其Mark as Resources。在该目录下添加**application.yml**文件。

## 2.Pom依赖
创建好项目后，IDEA默认会打开pom文件，如果没打开，请找到项目根目录下的pom.xml文件。

现在我们将需要的依赖写入pom文件中。需引入的依赖如下
``` xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
</properties>
<dependencies>
    <!--因为JAXB-API是java ee的一部分，在jdk10中没有在默认的类路径中；-->
    <!--java ee api在jdk中还是存在的，默认没有加载而已，jdk9中引入了模块的概念，可以使用-->
    <!--模块命令–add-modules java.xml.bind引入jaxb-api;-->
    <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
        <version>2.3.0</version>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.bind</groupId>
        <artifactId>jaxb-impl</artifactId>
        <version>2.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.glassfish.jaxb</groupId>
        <artifactId>jaxb-runtime</artifactId>
        <version>2.3.0</version>
    </dependency>
    <dependency>
        <groupId>javax.activation</groupId>
        <artifactId>activation</artifactId>
        <version>1.1.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
        <scope>true</scope>
    </dependency>
        <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
            <!-- 没有该配置，devtools 不生效 -->
            <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```
此处主要引入三个依赖：spring-boot-starter-web，spring-boot-starter-actuator，spring-boot-devtools。下文分别对其作用进行介绍。

注：粗体部分为Java8和Java10的区别，如果您使用的是Java8请去除该部分依赖。本章即使使用的是Java10也可以去除此部分依赖。

## 3.创建启动类
新增demo包，并将默认创建的App类删除（测试类也删除）。创建自己的启动类，例如：
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args){
        SpringApplication.run(DemoApplication.class,args);
    }
}
```
至此，一个Spring Boot 2 项目已经创建完成了。点击右上角绿色的运行或者调试按钮即可运行起来。IEDA底部会出现一个控制台，并且可以看到spring boot图形输出，如无报错信息，控制台信息输出完毕之后项目及运行成功。

虽然运行起来了，但是除了控制台，其他并没有任何效果。莫急。

前文说到，我们引用了**spring-boot-starter-web**，这是一个对Web全栈的开发支持，包含Tomcat和spring-webmvc。下一步我们就利用这个依赖创建一个controller。

## 4.创建controller
关闭项目。新增包controller并添加HelloController类。

HelloController类代码如下：
``` java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String helloworld() {
        return "hello,world!";
    }
}
```
再次运行，浏览器输入localhost:8080/hello即可看到浏览器中输出了hello,world字样(默认使用的是8080端口)。

## 5.热部署

热部署已经不是新鲜概念了。第4步在新增包和类的之后要关闭项目重新启动，否则新增或着修改源码不会对运行中的项目产生影响。热部署即是在不用关闭程序的前提下修改源代码可以自动重新部署项目。接下来我们动手实现。

pom.xml中引用的**spring-boot-devtools**即是实现热部署的关键组件。

在Intelijj IDEA上实现热部署需做一下简单的配置：
> 1 File->Settings界面，搜索Compiler
> 2 点击左侧的Compiler，勾选右侧**Build project automatically**
> 3 点击OK，应用配置
> 4 Ctrl+Shift+Alt+/ 打开maintenance界面
> 5 选择Registry…… 打开Registry界面
> 6 找到 **compiler.automake.allow.when.app.running**
> 7 勾选右侧的复选框
> 8 点击Close，关闭界面

修改HelloController里helloworld方法里的内容。如将返回内容改为hello world! I am spring boot!结果如下：
``` java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String helloworld() {
        return "hello,world! I am spring boot!";
    }
}
```
稍等片刻，控制台会再次看到spring boot的图形，项目重新部署。此时刷新浏览器即可看到变更后的内容。热部署实现！

## 6.日志配置

Spring Boot官方推荐使用logback。

接下来动手实现logback日志配置。

Spring Boot默认会加载 classpath:logback-spring.xml 或者 classpath:logback-spring.groovy。如需要自定义文件名称，在 application.yml 中配置 logging.config 选项即可。在 src/main/resources 下创建 logback-spring.xml 文件，内容如下：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 文件输出格式 -->
    <property name="PATTERN" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n"/>
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>${PATTERN}</pattern>
                <charset>UTF-8</charset>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="CONSOLE"/>
        </root>
</configuration>
```
修改HelloController类，添加一个日志输出。代码如下：
``` java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class HelloController {
    private final static Logger logger = LoggerFactory.getLogger(HelloController.class);
    @GetMapping("/hello")
    public String helloworld() {
        logger.info("收到了一个请求！");
        return "hello,world! I am spring boot!";
    }
}
```
当浏览器访问localhost:8080/hello时，控制台即会输出**收到了一个请求！**
此处的日志只是配置了控制台的输出，非常简单。即使不配置，默认也会有此效果。

## 7.运维与控制

微服务架构中一个很重要的特点是将原本庞大的单体系统拆分程多个提供不同服务的应用。加之同一个服务的高可用配置，应用会不断增多，系统的维护复杂度就会大大提升。我们需要实现一套自动化的监控运维机制，这套机制可以不间断的收集各个微服务应用的各项指标情况，如环境变量、垃圾收集信息、内存信息、线程池信息等。

我们的pom中引用的**spring-boot-starter-actuator**即是这样一个能够自动为Spring Boot构建的应用提供一系列用于监控的端点的模块。下面我们来一探究竟。

也许细心的读者可能发现在项目启动过程中，控制台会有一些含有actuator字样的输出。这些即为actuator暴露出的监控端点。

可以浏览器访问 localhost:8080/actuator查看效果

## 8.actuator原生端点

actuator默认只打开了少量的端点，我们可以通过配置来开启相应的端点。在resources目录下添加application.yml。这是Spring Boot默认读取的配置文件（也可以是application.properties、bootstrap.yml、bootstrap.properties等）。简单起见，我们在application.yml中将actuator所有端点都打开。配置如下：
``` yml
management:
    endpoints:
        web:
            exposure:
                include: "*"
```
这时，控制台会输出更多含有actuator字样的日志信息，刷新浏览器也会看到更多的信息显示出来。

这里显示actuator已经打开的所有的可用端点。我们选择beans端点，即可查看系统中已经创建的所有的bean(localhost:8080/actuator/beans)，其中也包含我们自己创建的HelloController类。

根据这些端点的作用，我们可以将原生端点分为以下三大类：

- 应用配置类：获取应用程序中加载的应用配置、环境变量、自动化配置报告等与Spring Boot应用密切相关的配置类信息；
- 度量指标类：获取应用程序运行过程中用于监控的度量指标，如内存信息、线程池信息、HTTP请求统计等；
- 操作控制类：提供了对应的关闭等操作类功能。

如果您对这些端点比较感兴趣可以访问本章开头提供的Spring Boot官方文档来了解更详细的解读。在此我举一个通过浏览器关闭应用程序的例子来加深您对actuator端点作用的认识。

首先要将shutdown端点开启，同样在application.yml文件中新增一句配置，修改后内容如下：
``` yml
management:
    endpoints:
        web:
            exposure:
                include: "*"
    endpoint:
        shutdown:
            enabled: true
```
由于shutdown端点需要使用POST方式请求，此时您需要借助api接口测试工具，此类工具较多（postman等），读者自行搜索，本章使用的是chrome浏览器的一个插件：Restlet Client。访问shutdown端点(localhost:8080/actuator/shutdown)即可关闭正在运行中的应用。

转到IDEA，此时应用已经为关闭状态。

## 9.单元测试

test下添加HelloControllerTest类，代码如下:
``` java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import static org.hamcrest.core.IsEqual.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;
    @Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
        .andExpect(status().isOk())
        .andExpect(content().string(equalTo("hello,world! I am spring boot!")));
    }
}
```
在getHello方法中右键选择run 'getHello()'即可运行，如无特殊原因，单元测试将会成功完成。

## 10.多环境配置

我们都会遇到以下的场景：当我们在开发阶段，使用自己的机器开发，测试的时候需要用的测试服务器测试，上线时使用正式环境的服务器。这三种环境需要的配置信息都不一样，当我们切换环境运行项目时，需要手动的修改多处配置信息，非常容易出错。

为了解决上述问题，Spring Boot 提供多环境配置的机制，让开发者非常容易的根据需求而切换不同的配置环境。

修改application.yml文件：
``` yml
#公共部分
spring:
    application:
    name: spring-boot-demo(可自行指定)
#指定使用哪一个配置
profiles:
    active: dev
# 开发环境 使用8080端口，打开actuator全部端点和shutdown端点
---
server:
    port: 8080
spring:
    profiles: dev

management:
    endpoints:
        web:
            exposure:
                include: "*"
    endpoint:
        shutdown:
            enabled: true
#测试环境 使用8081端口，打开actuator全部端点和shutdown端点
---
server:
    port: 8081
spring:
    profiles: test

management:
    endpoints:
        web:
            exposure:
                include: "*"
    endpoint:
        shutdown:
            enabled: true
#生产环境 使用8082端口，使用actuator默认端点
---
server:
    port: 8082
spring:
    profiles: pro
```
修改profiles:active节点的值即可指定使用哪一个环境的配置。

同样我们也可以对日志进行多环境配置。在logback-spring.xml里的appender节点外使用springProfile将其包起即可实现多环境配置。这里以日志输出格式为例：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 文件输出格式 -->
    <property name="PATTERN" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n"/>
    <property name="TEST_PATTERN" value="%-12(%d{HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n"/>
    <property name="PROD_PATTERN" value="%-12(%d{HH:mm:ss}) |-%-5level [%thread] %c [%L] -| %msg%n"/>
    <!--开发环境使用完成的日期时间格式-->
    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>${PATTERN}</pattern>
                <!-- 控制台也要使用UTF-8，不要使用GBK，否则会中文乱码 -->
                <charset>UTF-8</charset>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
        <!--测试环境使用完整的时间格式-->
    <springProfile name="test">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>${TEST_PATTERN}</pattern>
                <!-- 控制台也要使用UTF-8，不要使用GBK，否则会中文乱码 -->
                <charset>UTF-8</charset>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    <!--生产环境，使用时分秒格式-->
    <springProfile name="prod">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>${PROD_PATTERN}</pattern>
                <!-- 控制台也要使用UTF-8，不要使用GBK，否则会中文乱码 -->
                <charset>UTF-8</charset>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```
分别运行起来后即可观察各个配置环境的运行效果。

## 11.打包

Spring Boot项目打包可分为两种，jar和war。

默认情况下，通过 maven 执行 package 命令后，会生成 jar 包，且该 jar 包会内置了 tomcat 容器，因此我们可以通过 java -jar 就可以运行项目。
> 1.打包：进入项目的根目录，按住shift右键选择“在此处打开powshell窗口”，打开后输入mvn package命令然后回车。
> 2.运行：进入target目录，同样按住shift右键选择“在此处打开powshell窗口”，打开后输入java -jar .\spring-boot-demo-1.0-SNAPSHOT.jar命令然后回车。和在IDEA一样，控制台台会有spring boot图形输出。
如果要打成war包，修改 pom.xml 文件，将 jar(默认) 改成 war，如下：

``` xml
<project>
    ....
    <packaging>war</packaging>
    ....
</project>
```

并且让DemoApplication类继承 SpringBootServletInitializer 并重写 configure 方法，如下：
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
/**
* 该注解指定项目为springboot，由此类当作程序入口自动装配 web 依赖的环境
**/
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(DemoApplication.class);
    }
}
```
这样即可将打包好的war包放入tomcat(8以上)中运行。

至此本章关于Spring Boot 2 的相关介绍全部结束。希望对你有所帮助。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。