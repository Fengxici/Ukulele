---
title: 构建生命周期
date: 2019-09-06 17:04:15
tags:
- Maven
- 生命周期
categories:
- 构建
- Maven
---

```flow
st=>start: 开始
op=>operation: validate
op=>operation: compile
op=>operation: test
op=>operation: package
op=>operation: verify
op=>operation: install
op=>operation: deploy
e=>end: 结束
```
|阶段|处理|描述|
|----|----|---|
|验证 validate|验证项目|验证项目是否正确且所有必须信息是可用的|
|编译 compile|执行编译|源代码编译在此阶段完成|
|测试 Test|测试|使用适当的单元测试框架（例如JUnit）运行测试|
|包装 package|打包|创建JAR/WAR包如在 pom.xml 中定义提及的包|
|检查 verify|检查|对集成测试的结果进行检查，以保证质量达标|
|安装 install|安装|安装打包的项目到本地仓库，以供其他项目使用|
|部署 deploy|部署|拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程|

Maven 有以下三个标准的生命周期：

> 1. clean：项目清理的处理
> 2. default(或 build)：项目部署的处理
> 3. site：项目站点文档创建的处理

# Clean 生命周期
当我们执行 
> mvn post-clean 

命令时，Maven 调用 clean 生命周期，它包含以下阶段：

- pre-clean：执行一些需要在clean之前完成的工作
- clean：移除所有上一次构建生成的文件
- post-clean：执行一些需要在clean之后立刻完成的工作

**mvn clean** 中的 clean 就是上面的 clean，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，如果执行 mvn clean 将运行以下两个生命周期阶段：**pre-clean, clean**
如果我们运行 mvn post-clean ，则运行以下三个生命周期阶段:**pre-clean, clean, post-clean**

# Default (Build) 生命周期
这是 Maven 的主要生命周期，被用于构建应用，包括下面的 23 个阶段：
| 生命周期阶段|描述|
|------------|----|
|validate（校验）|校验项目是否正确并且所有必要的信息可以完成项目的构建过程|
initialize（初始化）|初始化构建状态，比如设置属性值|
generate-sources（生成源代码）|生成包含在编译阶段中的任何源代码|
process-sources（处理源代码）|处理源代码，比如说，过滤任意值|
generate-resources（生成资源文件）|生成将会包含在项目包中的资源文件|
process-resources （处理资源文件）|复制和处理资源到目标目录，为打包阶段最好准备|
compile（编译）|编译项目的源代码|
process-classes（处理类文件）|处理编译生成的文件，比如说对Java class文件做字节码改善优化|
generate-test-sources（生成测试源代码）|生成包含在编译阶段中的任何测试源代码|
process-test-sources（处理测试源代码）|处理测试源代码，比如说，过滤任意值|
generate-test-resources（生成测试资源文件）|为测试创建资源文件|
process-test-resources（处理测试资源文件）|复制和处理测试资源到目标目录|
test-compile（编译测试源码）|编译测试源代码到测试目标目录|
process-test-classes（处理测试类文件）|处理测试源码编译生成的文件|
test（测试）|使用合适的单元测试框架运行测试（Juint是其中之一）|
prepare-package（准备打包）|在实际打包之前，执行任何的必要的操作为打包做准备|
package（打包）|将编译后的代码打包成可分发格式的文件，比如JAR、WAR或者EAR文件|
pre-integration-test（集成测试前）|在执行集成测试前进行必要的动作。比如说，搭建需要的环境|
integration-test（集成测试）|处理和部署项目到可以运行集成测试环境中|
post-integration-test（集成测试后）|在执行集成测试完成后进行必要的动作。比如说，清理集成测试环境|
verify （验证）|运行任意的检查来验证项目包有效且达到质量标准|
install（安装）|安装项目包到本地仓库，这样项目包可以用作其他本地项目的依赖|
deploy（部署）|将最终的项目包复制到远程仓库中与其他开发者和项目共享|

当一个阶段通过 Maven 命令调用时，例如 mvn compile，只有该阶段之前以及包括该阶段在内的所有阶段会被执行。

不同的 maven 目标将根据打包的类型（JAR / WAR / EAR），被绑定到不同的 Maven 生命周期阶段。

# Site 生命周期
Maven Site 插件一般用来创建新的报告文档、部署站点等。

- pre-site：执行一些需要在生成站点文档之前完成的工作
- site：生成项目的站点文档
- post-site： 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
- site-deploy：将生成的站点文档部署到特定的服务器上.

这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这是Maven相当强大的功能，Manager比较喜欢，文档及统计数据自动生成，很好看。