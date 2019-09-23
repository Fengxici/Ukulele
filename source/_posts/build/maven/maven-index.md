---
title: Maven 开篇
date: 2019-09-10 17:04:15
tags:
- Maven
categories:
- 构建
- Maven
---

Maven，是 Apache 下的一个纯 Java 开发的开源项目。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。可以对 Java 项目进行构建、依赖管理等。

由于Ukulele后台项目使用maven作为构建工具，为了方便大家使用，特在此处开设专栏介绍maven相关知识。


# Maven 功能
Maven 能够帮助开发者完成以下工作：
- 构建
- 文档生成
- 报告
- 依赖
- SCMs
- 发布
- 分发
- 邮件列表

# 约定配置
Maven 提倡使用一个共同的标准目录结构，Maven 使用**约定优于配置**的原则，大家尽可能的遵守这样的目录结构。如下所示：

|目录|说明|
|----|----|
|${basedir}|项目根目录，存放pom.xml和所有的子目录|
|${basedir}/src/main/java|项目的java源代码|
|${basedir}/src/main/resources|项目的资源，比如说property文件，springmvc.xml|
|${basedir}/src/test/java|项目的测试类，比如Junit代码|
|${basedir}/src/test/resources|测试用的资源|
|${basedir}/target|打包输出目录|
|${basedir}/target/classes|编译输出目录|
|${basedir}/target/test-classes|测试编译输出目录|

# 官方文档
更多maven相关知识请访问：https://maven.apache.org/users/index.html

# 安装
下载地址 ：https://maven.apache.org/download.cgi

Maven 是一个基于 Java 的工具，所以要做的第一件事情就是安装JDK。

Maven无需安装，官网下载的压缩包解压缩至固定的目录下即可。

# 配置
1. 添加环境变量 MAVEN_HOME
2. 编辑系统变量 Path，添加变量值：;%MAVEN_HOME%\bin
3. 修改本地仓库位置，
> 1. 找到maven根目录下的config/settings.xml文件
> 2. 找到localRepository节点，填写自己的本地仓库地址，如：D:\maven\repo

这里需要说明的是，我们将自己的硬盘整理好，资料归资料，娱乐归娱乐，软件归软件，笔者曾经见过很多开发同学的电脑硬盘一片混乱，文件随意存放，这是一个非常不好的习惯，会为你的开发工作造成不必要的麻烦。另外，文件夹名称尽量不要使用中文，虽然很对软件对中文路径的支持越来越友好，但仍然会有不支持的情况，为了避免造成不必要的麻烦，请务必尽量使用英文来明明文件夹及文件

Maven默认的本地仓库目录位置为~/.m2/repository，对应windows系统即为C:\Users\xxx\.m2\repository，随着引用的增长，这个目录会膨胀，系统盘会被占用殆尽(可能不会这么夸张)，因此我们应该合理规划本地仓库位置。
