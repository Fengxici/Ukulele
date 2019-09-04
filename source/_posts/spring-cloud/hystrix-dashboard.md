---
title: 第七章 服务熔断监控
date: 2019-08-31 10:23:05
tags: 
- spring cloud
- spring boot
- hystrix dashboard
categories:
- spring cloud
- 教程
---

> 实时监控Hystrix的各项指标信息。

闲言碎语不多讲，动手体验来瞧一瞧。

依然，我们是在消费者项目上进行改造。本章基于第六章进行升级。

## 1 创建新项目，命名为hystrix-dashboard-demo。
## 2 修改pom依赖。较上一章新增了hystrix-dashboard依赖。
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
```
## 3 启动类，同第六章。

## 4 controller包及HystrixController类，同第六章。

## 5 配置文件，同第六章。

## 6 最后一点调整。
spring boot 2.0.2之后需要注册HystrixMetricsStreamServlet。**启动类**中添加如下代码：
``` java
@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean<>(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```
## 7 OK。至此全部项目创建完毕，启动起来看看效果吧。

> 1 启动注册中心
> 2 启动服务提供者
> 3 启动本项目
> 4 浏览打开localhost:9090/ribbon/get备用
> 5 浏览器打开localhost:9090/hystrix，您将看到一只可爱的小熊
> 6 在最长的输入框中输入:localhost:9090/hystrix.stream，然后点击Monitor Stream按钮
> 7 多次刷新第4打开的地址，观察hystrix dashboard界面的变化

可以试着关闭提供者服务再刷新**4**打开的地址观察下hysrix dashboard界面的变化。

好了，本章关于hystrix dashboard的介绍就到这里。下一章将会介绍hystrix dashboard的集群模式——Turbine。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。