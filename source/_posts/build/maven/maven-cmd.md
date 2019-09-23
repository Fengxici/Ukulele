---
title: 第八章 一些常用的Maven命令
date: 2019-09-18 17:04:15
tags:
- Maven
- cmd
categories:
- 构建
- Maven
---

编译
> mvn compile

打包
> mvn package

安装
> mvn install

部署
> mvn deploy

安装jar包到本地仓库
> mvn install:install-file -Dfile=jar包的位置 -DgroupId=groupId -DartifactId=artifactId -Dversion=version -Dpackaging=jar

安装jar包到远程仓库
mvn deploy:deploy-file -Dfile=jar包的位置 -DgroupId=groupId -DartifactId=artifactId -Dversion=version -Dpackaging=jar