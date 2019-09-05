---
title: 源码使用说明
p: spring-cloud/demo-source
date: 2019-09-05 11:25:55
tags:
- spring cloud
- 源码
categories:
- spring cloud
- 教程
---

# 前言

本篇汇总了前12篇文章的源码，并对其使用方法做出说明。读者可与自己的实现做个比较。

之所以未在每章中列出源码链接，主要是想让读者能都动手去实现，并且积极思考和解决在实现过程中所出现的问题。编程工作是一个实践性很强的工作，只有勤动手勤思考才能阵阵掌握所学技能。

***请各位读者务必动手做完第二章-第十一章的编码工作，再来看下面的内容，否则您将不会有太多收获！！！***

---

# 源码地址

- github:
> https://github.com/Fengxici/spring-cloud-current-demo

- gitee:
> https://gitee.com/fengxici/spring-cloud-currect-demo

# 目录结构

- config-demo --------------------第十一章 配置中信
    - config-client ----------------------配置中心客户端
    - config-server ----------------------配置中信服务端
- consumer-service ---------------第四章 消费者服务
- eureka-demo --------------------第二章 注册中心Eureka
    - eureka-client ----------------------注册中心客户端
    - eureka-client ----------------------注册中心服务端
- gateway-demo -------------------第五章 网关
    - cloud-gateway-demo -----------------Spring Cloud Gateway
    - zuul-demo --------------------------Netflix Zuul
- hystrix-dashboard-demo ---------第七章 服务熔断监控
- hystrix-demo -------------------第六章 服务熔断
- monitor-demo -------------------第十章 配置中心
    - client-demo ------------------------配置中心客户端
    - server-demo ------------------------配置中心服务端
- provider-service ---------------第四章 服务消费者
- spring-boot-demo ---------------第二章 Spring Boot简介
- trace-demo ---------------------第九章 链路跟踪
    - customer-demo ----------------------服务消费者
    - provider-demo ----------------------服务提供者
    - sleuth-zipkin-demo -----------------链路跟踪服务
- turbine-demo ---------------------------第八章 服务熔断汇总

# 章节介绍
每个章节的知识点不同，所要用到的命令以及各章节之间的关联都有所不同，下面依次按章节进行介绍。在此之前，先把可能要用到的命令罗列如下：
  - 打包 
  > mvn package

  - 运行(maven方式)
  > mvn spring-boot:run

以上需要在pom.xml所在目录下执行

  - 运行(java方式)
  > java -jar xxxx.jar

  - 多环境运行(java方式)
  > java jar xxx.jar --spring.profiles.active=dev

  - 修改服务启动端口(java方式)
  > java jar xxx.jar --server.port=9090

以上需要在.jar文件所在目录下执行

## 第一章
使用idea导入项目时，请将导入目录指定至spring-boot-demo下

非开发工具(如IDEA)下我们可以这样运行程序，在项目根目录下（pom.xml所在目录)
   > 1 打包 mvn package
   > 2 运行 mvn spring-boot:run 

或
   > 1 打包 mvn package
   > 2 进入target目录 cd target
   > 3 运行 java jar spring-boot-demo-1.0-SNAPSHOT.jar

若需要修改端口第2步改为
   > java -jar spring-boot-demo-1.0-SNAPSHOT.jar --server.port=9090

## 第二章
使用idea导入项目时，请将导入目录指定至**eureka-demo**下

本章代码是最终完成版，eureke-server/src/main/resources目录下的application.yml配置文件里放置的集群配置。如需单机版运行，请参照注释说明开启单机版配置，注释集群版注释。

本章含两个子项目，必须先启动eureka服务端(eureka-server)，再启动eureka客户端(erureka-client).

打包，运行方式同第一章

集群部署请先修改好eureka-server下的application.yml的配置，后依次执行：
   > java -jar eureka-server.jar --spring.profiles.active=peer1

   > java -jar eureka-server.jar --spring.profiles.active=peer2

   > java -jar eureka-server.jar --spring.profiles.active=peer3

## 第三章
使用idea导入项目时，请将导入目录指定至**provider-service**下

本章需使用第二章的eureka-server模块作为注册中心，并且使用的是单机版，所以请先修改好eureka-server的配置。建议改好后打包成jar包，并复制到其他目录，后续项目几乎都会用到该注册中心。

需先启动注册中心，再启动提供者服务，所使用命令与上文相同。

## 第四章
使用idea导入项目时，请将导入目录指定至**consumer-service**下

本章同样使用第二章的eureka-server模块作为注册中心，在此不再赘述。

本章还要使用第三章的提供者服务，所有在启动本章服务之前，先启动第三章的服务。

一切正常的情况下，关闭第三章的提供者服务，观察效果。

## 第五章
使用idea导入项目时，请将导入目录指定至**getway-demo**下

本章的两个服务相互独立，无相互依赖关系，无需关注前后顺序。两个项目单独实验。

本章同样使用了注册中心，服务提供者，服务消费者，请依次启动这三个服务。**建议将他们打包放置在固定的目录下，以备后续使用**

打包或启动所使用命令均与前文相同。

## 第六章
使用idea导入项目时，请将导入目录指定至**hystrix-demo**下

熔断相对于服务消费者而言，如果获取不到服务提供者的调用，服务消费者要进行降级处理，所以本章是对第四章服务消费者的改造，因此本章使用到的项目有 第二章(eureka-server)和第三章(provider-service)。 请先依次启动这两个服务。

本章内容可以第四章做个对比，从中观察熔断的作用。

## 第七章
使用idea导入项目时，请将导入目录指定至**hystrix-dashboard-demo**下

本章与第六章相似。启动过程和依赖关系都同第六章，在此不再赘述。

相比于第六章本章在效果上多了熔断监控界面，在这个界面里可以很直观的观测到消费者对提供者的调用情况。

## 第八章
使用idea导入项目时，请将导入目录指定至**turbine-demo**下

本章是对第七章服务监控的汇总。当有很多消费者的时候，我们不可能去每一个消费者那里查看调用情况，甚至也不能知道到哪个消费者那里去查看，因此，需要一个集中查看调用健康状态的地方。

本章依赖的项目有 第二章(eureka-server)，第三章(provider-service)和第七章（hystrix-dashbaord-service)，请先依次启动他们。

需要说明的是hystrix-dashbaord-service需要启动至少两个才能看出效果。启动两个以上的服务只需修改端口号即可。所以启动命令可使用
  > java -jar hystrix-dashboard-service.jar --server.port=10000

  > java -jar hystrix-dashboard-service.jar --server.port=20000

## 第九章
使用idea导入项目时，请将导入目录指定至**trace-demo**下

本章需要依赖第三章和第四章项目，因为改动部分代码，所以在这里进行了重新创建。

注册中心服务需要先启动，其他无先后顺序。

## 第十章
使用idea导入项目时，请将导入目录指定至**monitor-demo**下

注册中心服务需要先启动，其他无先后顺序。

## 第十一章
使用idea导入项目时，请将导入目录指定至**config-demo**下

注册中心服务需要先启动，其他无先后顺序。

本章默认将配置文件放在classpath下，所以打包后运行就不能修改配置文件，因此请使用mvn方式运行。要修改的配置文件在target/classes/config目录下。

看过文章依然存疑的读者请移步源码仓库，扫描二维码进群讨论。

如果您觉得对您有帮助请给项目一个star。

创作不易，请君鼓励。谢谢。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。





