---
title: 第六章 Maven 插件
date: 2019-09-16 17:04:15
tags:
- Maven
- plugin
categories:
- 构建
- Maven
---
如前文所述，Maven 有以下三个标准的生命周期：

- clean：项目清理的处理
- default(或 build)：项目部署的处理
- site：项目站点文档创建的处理

每个生命周期中都包含着一系列的阶段(phase)。这些 phase 就相当于 Maven 提供的统一的接口，然后这些 phase 的实现由 Maven 的插件来完成。

我们在输入 mvn 命令的时候 比如 mvn clean，clean 对应的就是 Clean 生命周期中的 clean 阶段。但是 clean 的具体操作是由 maven-clean-plugin 来实现的。

所以说 Maven 生命周期的每一个阶段的具体实现都是由 Maven 插件实现的。

Maven 实际上是一个依赖插件执行的框架，每个任务实际上是由插件完成。Maven 插件通常被用来：
- 创建 jar 文件
- 创建 war 文件
- 编译代码文件
- 代码单元测试
- 创建工程文档
-创建工程报告

插件通常提供了一个目标的集合，并且可以使用下面的语法执行：

> mvn [plugin-name]:[goal-name]

例如，一个 Java 工程可以使用 maven-compiler-plugin 的 compile-goal 编译，使用以下命令：

> mvn compiler:compile

# 插件类型
Maven 提供了下面两种类型的插件：
- Build plugins:在构建时执行，并在 pom.xml 的 元素中配置。
- Reporting plugins:在网站生成过程中执行，并在 pom.xml 的 元素中配置。

一些常用的插件
- clean	构建之后清理目标文件。删除目标目录。
- compiler	编译 Java 源文件。
- surefile	运行 JUnit 单元测试。创建测试报告。
- jar	从当前工程中构建 JAR 文件。
- war	从当前工程中构建 WAR 文件。
- javadoc	为工程生成 Javadoc。

