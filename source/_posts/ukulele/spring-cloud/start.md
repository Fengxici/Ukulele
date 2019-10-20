---
title: 第一章 Spring Cloud 系列微服务启动
date: 2019-10-16 17:04:15
tags:
- 清单
- 部署
- 开发
categories:
- Ukulele
- Spring Cloud
---
# 前言
Ukulele全栈开发框架所规划的服务端皆采用微服务架构。

说到微服务，JAVA体系下最流行的非Spring Cloud家族莫属了。Ukulele精心挑选了Spring Cloud中常用的组建和库，将其组织成一个微服务所需的基础服务、系统服务，并对部分库进行了二次封装以方便开发使用。这些项目清单如下：
<escape>
<table>
<thead>
<tr><td> 项目</td><td>描述</td><td>子项目</td><td>链接</td></tr>
</thead>
<tbody>
<tr><td rowspan="4">ukulel-boot</td><td rowspan="4">spring cloud微服务基础组件</td><td>注册中心(register)</td><td rowspan="4"> <a href="https://github.com/Fengxici/Ukulele-Boot">https://github.com/Fengxici/Ukulele-Boot</a></td></tr>
<tr><td>监控（monitor)</td></tr>
<tr><td>熔断（circuit) </td></tr>
<tr><td>链路（trace）</td></tr>
<tr><td rowspan="10">ukulele-master</td><td rowspan="10">ukulele公共库项目</td><td>通用组件（ukulele-commom）</td><td rowspan="10"><a href="https://github.com/Fengxici/Ukulele-Master">https://github.com/Fengxici/Ukulele-Master</a></td></tr>
<tr><td>zookeeper分布式锁集成（ukulele-curator）</td></tr>
<tr><td>网络库（ukulele-http）</td></tr>
<tr><td>日志库（ukulele-log）</td></tr>
<tr><td>持久层库（ukulele-persistence）</td></tr>
<tr><td>redis分布式锁集成（ukulele-redisson）</td></tr>
<tr><td>ribbon集成（ukulele-ribbon）</td></tr>
<tr><td>swagger集成（ukulele-swagger）</td></tr>
<tr><td>web层集成库（ukulele-web）</td></tr>
<tr><td>Eureka节点启停管理（ukulele-eureka)</td></tr>
<tr><td rowspan="4">ukulele-data</td><td rowspan="4">ukulele传输层库</td><td>auth数据（Auth-Data）</td><td rowspan="4"><a href="https://github.com/Fengxici/Ukulele-Data">https://github.com/Fengxici/Ukulele-Data</a></td></tr>
<tr><td>系统管理数据（Portal-Data)</td></tr>
<tr><td>用户管理数据（User-Data)</td></tr>
<tr><td>访问日志数据（syslog-data)</td></tr>
<tr><td rowspan="8">ukulele-facade</td><td rowspan="8">ukulele接口库</td><td>Auth接口（Auth-Cloud-Facade）</td><td rowspan="8"><a href="https://github.com/Fengxici/Ukulele-Facade">https://github.com/Fengxici/Ukulele-Facade</a></td></tr>
<tr><td>Auth接口（Auth-Dubbo-Facade)</td></tr>
<tr><td>系统接口（Portal-Cloud-Facade)</td></tr>
<tr><td>系统接口（Portal-Dubbo-Facade)</td></tr>
<tr><td>用户接口（User-Cloud-Facade)</td></tr>
<tr><td>用户接口（User-Dubbo-Facade)</td></tr>
<tr><td>访问日志接口（Syslog-Cloud-Facade)</td></tr>
<tr><td>访问日志接口（Syslog-Cloud-Facade)</td></tr>
<tr><td rowspan="5">ukulele-cloud</td><td rowspan="5">ukulele服务</td><td>Auth服务（Ukulele-Auth）</td><td rowspan="5"><a href="https://github.com/Fengxici/Ukulele-Cloud">https://github.com/Fengxici/Ukulele-Cloud</a></td></tr>
<tr><td>网关服务（Ukulele-Gateway)</td></tr>
<tr><td>系统服务（Ukulele-Portal)</td></tr>
<tr><td>用户服务（Ukulele-User)</td></tr>
<tr><td>访问日志服务（Ukulele-Syslog)</td></tr>
</tbody>
</table>
</escape>

# 项目介绍
## ukulele-boot
Ukulele将Spring Cloud微服务中几乎不会变动的服务放在一个单独的项目中。一来在一个组织中，这些公共服务部署之后很少变动的，业务开发人员不必关心他们的内部细节；二来避免了将不必要的服务放在同一个项目下，减少整个项目的构建时间。[有点单一职责的意思]

如上列表所示，这些基础服务包含：注册中心，链路，监控及熔断。这些都是微服务的基础设施。

### 注册中心
端口使用 **8080** 用户名及密码：**root/root** 可在配置文件resources/application.yml中进行修改。暂未使用高可用模式，如有需要请自行添加。访问地址为：http://ip:port

启动方式
1. maven方式
> 在register目录下执行 **mvn spring-boot:run** 命令即可启动
2. jar方式
> - 在register目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar register-1.0-SNAPSHOT.jar** 即可启动

**注册中心必须启动，其他所有服务都会注册到注册中心**

![注册中心登陆界面](/images/ukulele/spring-cloud/register.png)
![注册中心面板](/images/ukulele/spring-cloud/register2.png)

### 监控
端口 **5050** 用户名及密码 **root/root** 可在配置文件resources/application.yml中进行修改。**若注册中心的端口发生变更，此处的配置也必须要进行对应的修改**。访问地址为：http://ip:port

启动方式
1. maven方式
> 在monitor目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在monitor目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar monitor-1.0-SNAPSHOT.jar** 即可启动

**监控服务可选，如果您的硬件资源紧张可以选择不启动，不影响项目运行**
![监控登陆界面](/images/ukulele/spring-cloud/monitor.png)
![监控面板](/images/ukulele/spring-cloud/monitor2.png)
### 熔断
端口 **6060** 用户名及密码 **root/root** 可在配置文件resources/application.yml中进行修改。**若注册中心的端口发生变更，此处的配置也必须要进行对应的修改**。访问地址为：http://ip:port/hystrix 注意末尾的**hystrix**

启动方式
1. maven方式
> 在monitor目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在monitor目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar circuit-1.0-SNAPSHOT.jar** 即可启动

**熔断服务可选，如果您的硬件资源紧张可以选择不启动，不影响项目运行**
![熔断面板](/images/ukulele/spring-cloud/circuit.jpg)
### 链路
端口 **7070** 用户名及密码 **root/root** 可在配置文件resources/application.yml中进行修改。**若注册中心的端口发生变更，此处的配置也必须要进行对应的修改**。访问地址为：http://ip:port

启动方式
1. maven方式
> 在monitor目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在monitor目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar trace-1.0-SNAPSHOT.jar** 即可启动

**链路服务可选，如果您的硬件资源紧张可以选择不启动，但需要将其他服务里的链路相关配置关掉**
![链路面板](/images/ukulele/spring-cloud/trace.png)

至此，ukulele基础组建已经启动完毕。如果您启动了所有服务，回看注册中心和监控会有如下效果
![础组建注册成功后注册中心基面板](/images/ukulele/spring-cloud/register3.jpg)
![基础组建完成后监控面板](/images/ukulele/spring-cloud/monitor3.png)
## ukulele-master
此项目经spring boot常用的一些组建进行了二次封装，以便于组织中的成员使用，甚至可以达到编码规范的作用。

此项目中各个模块的功能将会在后续章节中介绍。

安装
> 项目根目录执行 mvn install

## ukulele-data
Ukulele的系统服务包括权限服务(auth-service)、门户服务(portal-service)、系统日志服务(syslog-service)和用户服务(user-service)。客户端（浏览器、移动设备、桌面设备等）调用服务接口，或者服务之间相互调用接口会传输相关数据，此项目即是提供统一数据格式的项目。

安装
> 项目根目录执行 mvn install

## ukulele-facade
如上节所述，Ukulele系统服务包含四个服务，服务之间相互调用的接口汇总在该项目中(实际是所有接口)。每个服务均包含Spring Cloud系的Feign接口和Dubbo系的接口(后期将会调整)。

个人认为在实际的开发中，每个服务由一组单独的人员维护，其他无关人员不能直接接触服务相关代码，只能通过接口的方式调用。那么其实每个服务的facade和data应该是在同一个项目中的，Ukulele之所以将四个服务的data和facade拆分开各自放在一个项目中是因为:1.节省项目数量；2.data的引用场景更多(服务之间、安卓端)；

以上都是废话，我想怎么组织就怎么组织，你也可以（没有标准，自己觉得合适、方便即可）。

安装
> 项目根目录执行 mvn install

## 中间件
### 数据库
数据库文件位于ukulele-cloud根目录下**database/ukulele.sql**。这里采用的是四个服务单独建库。
### redis
默认配置
### rabbitmq
默认配置
## ukulele-cloud
该项目下含五个服务：网关服务(ukulele-gateway)、权限服务(ukulele-auth)、门户服务(ukulele-portal)、系统日志服务(ukulele-syslog)和用户服务(ukulele-user)。

### ukulele-auth
 用户登录，第三方授权等。端口 **9090** 

启动
1. maven方式
> 在Ukulele-Auth目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在Ukulele-Auth目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar Ukulele-Auth-1.0-SNAPSHOT.jar** 即可启动

访问：http://ip:port/auth 您将看到登陆界面

![权限服务登陆界面](/images/ukulele/spring-cloud/auth.jpg)

### ukulele-portal
系统配置。端口 **10101**

启动
1. maven方式
> 在Ukulele-Portal目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在Ukulele-Portal目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar ukulele-portal-1.0-SNAPSHOT.jar** 即可启动

访问 http://ip:port/swagger-ui.html 您将看到接口清单界面
![门户服务接口清单界面](/images/ukulele/spring-cloud/portal-swagger.png)
### ukulele-syslog
用户操作日志。端口 **20202**

启动
1. maven方式
> 在Ukulele-Syslog目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在Ukulele-Syslog目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar Ukulele-Syslog-1.0-SNAPSHOT.jar** 即可启动

### ukulele-user
系统用户服务。端口 **30303**

启动
1. maven方式
> 在Ukulele-User目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在Ukulele-User目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar Ukulele-User-1.0-SNAPSHOT.jar** 即可启动

访问 http://ip:port/swagger-ui.html 您将看到接口清单界面
![用户服务接口清单界面](/images/ukulele/spring-cloud/user-swagger.png)

### ukulele-gateway
系统用户服务。端口 **10000**

启动
1. maven方式
> 在Ukulele-Gateway目录下执行 **mvn spring-boot:run** 命令即可启动

2. jar方式
> - 在Ukulele-Gateway目录下执行**mvn package**将项目打包
> - 进入target目录，执行 **java -jar Ukulele-Gateway-1.0-SNAPSHOT.jar** 即可启动

**gateway服务启动前需先启动auth服务，其他无先后顺序要求**。因网关中采用的是serviceId来路由服务的，注册中心需要一点点时间来发现服务，所以建议gateway服务最后启动，利用gateway启动的时间差，等它好之后可以立即使用了。

至此所有项目启动完成。

访问 http://ip:port/swagger-ui.html 您将看到汇总其他服务的接口清单界面
![用户服务接口清单界面](/images/ukulele/spring-cloud/gateway-swagger.png)

至此所有项目启动完成。此时在回看注册中心和监控界面，将是如下效果
![所有服务注册中心界面](/images/ukulele/spring-cloud/register4.png)
![所有服务监控界面](/images/ukulele/spring-cloud/monitor4.png)

ukulele的angular版前端界面如下
![angular版登陆界面](/images/ukulele/spring-cloud/angular-login.png)
![angular版管理界面](/images/ukulele/spring-cloud/angular-manage.png)

本博客也会有相关文章对前端界面进行详细介绍，欢迎关注。

在此也感谢优秀的[ng-alain](https://ng-alain.com/zh)项目。

# 开发
各项目以maven方式导入开发工具。若您不了解maven建议您先去官网学习，或者在本博客maven章节中快速浏览。如果您不了解Spring Cloud，同样建议您先去官网学习，当然，本博客已有相关文章可供快速实践。

ukulele-master、ukulele-data、ukulele-facade三个库项目如果有变更，请在各自项目根路径下运行**mvn install**安装至本地仓库。如此，ukulele-cloud项目才能引用到。

如您要开发新服务，请参考portal、user或者syslog（俗称依样画葫芦）。

# 奉献  感恩  开放 勇敢 持续进步
