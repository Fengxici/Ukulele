---
title: Hello Ukulele
tags:
- 项目介绍
- 项目清单
categories:
- Ukulele
- 项目简介
---
欢迎来到 [Ukulele](https://fengxici.github.io/)! 

本项目的创建旨在打造一个企业级全栈开发框架，为中小企业降低开发难度。

Ukulele是一个乐器，它小巧、携带方便，声音好听可爱，能够激发节奏潜能。只要它在手中，没有你不会弹的歌。这正如我想要打造的企业级全站开发框架一样：小巧，易构建，对用户友好，涵盖绝大数企业开发需求。因此我将框架命名为 **Ukulele**.

Ukulele由一系列项目构成，概括起来即为
> 1. 后端采用微服务架构，支持两种主流开发语言：C#和JAVA
> 2. JAVA微服务支持Spring Cloud和Dubbo两种主流生态
> 3. 支持两中前端主流行框架Angular和Vue
> 4. 支持两种移动端主流系统的原生开发：Android和IOS
> 5. 支持三种桌面主流系统原生开发：Windows、Mac和Linux
> 6. 支持微信小程序

# 项目清单

## **1.Spring Cloud 系列**
<escape>
<table>
<thead>
<tr><td> 项目</td><td>描述</td><td>子项目</td><td>链接</td></tr>
</thead>
<tbody>
<tr><td rowspan="4">ukulel-boot</td><td rowspan="4">spring cloud微服务基础组建</td><td>注册中心(register)</td><td rowspan="4"> <a href="https://github.com/Fengxici/Ukulele-Boot">https://github.com/Fengxici/Ukulele-Boot</a></td></tr>
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


## **2.Dubbo系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukelele-Dubbo|


## **3.DotNet Core微服务系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukulele-Sharp|


## **4.前端系列**
|项目|描述|链接|
|----|----|---|
|ukulele-angualr|基于angular框架，以[ng-alain](https://ng-alain.com)为基础|https://github.com/Fengxici/Ukulele-Angular|
|ukulele-vue|给予vue框架，以[d2admin](https://doc.d2admin.fairyever.com)为基础(规划中)|https://github.com/Fengxici/Ukulele-Vue|


## **5.安卓系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukulele-Android|


## **6.IOS系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukulele-IOS|


## **7.Windows系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukulele-Win|


## **8.Linux系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukelele-Linux|


## **9.MAC系列**
|项目|描述|子项目|链接|
|----|----|-----|----|
|（规划中）|（规划中）|（规划中）|https://github.com/Fengxici/Ukelele-Mac|


## **10.微信小程序系列**
|项目|描述|链接|
|----|----|----|
|（规划中）|（规划中）|https://github.com/Fengxici/Ukulele-WeChatApp|

------------------

# 奉献  感恩  开放 勇敢 持续进步
