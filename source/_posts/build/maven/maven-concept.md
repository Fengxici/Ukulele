---
title: 第一章 相关概念
date: 2019-09-11 14:04:15
tags:
- Maven
- 概念
categories:
- 构建
- Maven
---

POM( Project Object Model，项目对象模型 ) 是 Maven 工程的基本工作单元，是一个XML文件，包含了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。

**pom.xml是玩转maven最重要的文件**

执行任务或目标时，Maven 会在当前目录中查找 POM。它读取 POM，获取所需的配置信息，然后执行目标。

POM 中可以指定以下配置：
- 项目依赖
- 插件
- 执行目标
- 项目构建 profile
- 项目版本
- 项目开发者列表
- 相关邮件列表信息

所有 POM 文件都需要 project 元素和三个必需字段：groupId，artifactId，version。

|节点|说明|
|----|----|
|project|工程的根标签|
|modelVersion|模型版本|
|groupId|这是工程组的标识。它在一个组织或者项目中通常是唯一的|
|artifactId|这是工程的标识。它通常是工程的名称|
|version|这是工程的版本号。在 artifact 的仓库中，它用来区分不同的版本|

# 父（Super）POM
父（Super）POM是 Maven 默认的 POM。所有的 POM 都继承自一个父 POM（无论是否显式定义了这个父 POM）。父 POM 包含了一些可以被继承的默认设置。因此，当 Maven 发现需要下载 POM 中的 依赖时，它会到 Super POM 中配置的默认仓库 http://repo1.maven.org/maven2 去下载。

Maven 使用 effective pom（Super pom 加上工程自己的配置）来执行相关的目标，它帮助开发者在 pom.xml 中做尽可能少的配置，当然这些配置可以被重写。

使用以下命令来查看 Super POM 默认配置：
> mvn help:effective-pom

effective-pom结果中，你可以看到 Maven 在执行目标时需要用到的默认工程源码目录结构、输出目录、需要的插件、仓库和报表目录。

Maven 的 pom.xml 文件也不需要手工编写。

Maven 提供了大量的原型插件来创建工程，包括工程结构和 pom.xml。

# 坐标
**Maven坐标**为各种构件引入了秩序，任何一个构件都必须明确定义自己的坐标，而一组Maven坐标是通过一些元素定义的，他们是groupId、artifactId、version、packaging、classifier。先看一组坐标定义，如下：
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>Greenwich.SR2</version>
</dependency>
```
## 坐标元素解释

> groupId：定义当前maven项目隶属的实际项目

> artifactId：定义实际项目中的一个Maven项目的模块

> version：定义Maven项目当前所处的版本

注意：Maven定义了一套完成的版本概念，以及快照SnapShot的概念

> packaging：定义Maven项目打包的方式（jar包、war包）
打包方式会影响到构建的生命周期，比如jar打包和war打包会使用不同的命令。当不定义packaging的时候，Maven 会使用默认值jar。

> classifier：用来帮助定义构建输出的一些附属构件，如Java文档和源代码。

注意：不能直接定义项目的classifier，因为附属构件不是项目直接默认生成的，而是由附加的插件帮助生成的。

## 坐标注意点
上述5个元素中： groupId、artifactld、 version 是必须定义的；packaging是可选的(默认为jar)；而classifier是不能直接定义的。

同时，项目构件的文件名是与坐标相对应的，一般的规则为artifactld-version[ -classifier].packaging  其中[ -classifier]表示可选。要强调的一点是：       packaging并非一定与构件展名对应，比如packaging为maven-plugin的构件扩展名为jar。

# 依赖
## 依赖的配置
一个依赖声明可以包含如下的一些元素：
``` xml
<dependency>
    <groupId>***</groupId >
    <artifactId>***</artifactId>
    <version>***</version>
    <type>***</type>
    <scope>***</scope>
    <optional>***</optional>
    <exclusions>
        <exclusison>
        ...
        </exclusion >
    ...
    </exclusions >
</dependency >
```
根元素projet下的一个或者多个dependency 元素需包含在一个**dependencies**标签下，以声明一个或者多个项目依赖。每个依赖可以包含的元素有:
> 1. groupld、artifactld和version：依赖的基本坐标，对于任何个依赖来说，基本坐标是最重要的，Maven 根据坐标才能找到需要的依赖。
> 2. type：依赖的类型，对应于项目坐标定义的packaging。大部分情况下，该元素不必声明，其默认值为jar
> 3. scope：依赖的范围
> 4. optional：标记依赖是否可选
> 5. exclusions：用来排除传递性依赖

大部分依赖声明只包含基本坐标，然而在一些特殊情况下，其他元素至关重要.

PS: 开发过程中我们可以通过：https://search.maven.org/ 来查询我们所需依赖的版本。

## 依赖的范围
依赖范围就是用来控制依赖与三种classpath ( 编译classpath、测试classpath、运行classpath)的关系。maven有以下几种依赖范围:
> compile（默认的）

编译依赖范围。如果没有指定，就会默认使用该依赖范围。使用此依赖范围的Maven依赖，对于编译、测试、运行三种classpath都有效。典型的例子是spring-core,在编译、测试和运行的时候都需要使用该依赖。

> test

测试依赖范围。使用此依赖范围的Maven依赖，只对于测试classpath有效，在编译主代码或者运行项目的使用时将无法使用此类依赖。典型的例子是JUnit,它只有在编译测试代码及运行测试的时候才需要。

> provided

已提供依赖范围。使用此依赖范围的Maven依赖，对于编译和测试classpath有效，但在运行时无效。典型的例子是:servlet-api,编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，就不需要Maven重复地引入一遍。

>runtime

运行时依赖范围。使用此依赖范围的Maven依赖，对于测试和运行classpath有效，但在编译主代码时无效。典型的例子是JDBC驱动实现，项目主代码的编译只需要JDK提供的JDBC接口，只有在执行测试或者运行项目的时候才需要实现上述接口的具体JDBC驱动。

> system

系统依赖范围。该依赖与三种classpath的关系，和provided依赖范围完全致。但是，使用system范围的依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，因此应该谨慎使用。systemPath 元素可以引用环境变量

> import

导人依赖范围,Maven 2.0.9及以上版本支持。该依赖范围不会对三种classpath产生实际的影响

## 依赖范围与classpath的关系
|依赖范围（Scope）|对编译classpath有效|对测试classpath有效|对运行时classpath有效|
|----------------|------------------|------------------|--------------------|
|compile|Y|Y|Y|
|test|-|Y|-|
|provide|Y|Y|-|
|runtime|-|Y|Y|
|system|Y|Y|-|

## 传递性依赖
> 假设A依赖于B, B依赖于C,我们说A对于B是第一直接依赖，B对于C是第二直接依赖，A对于C是传递性依赖。

> 第一直接依赖的范围和第二直接依赖的范围决定了传递性依赖的范围

||compile|test|provide|runtime|
|-|------|----|-------|-------|
|compile|compile|-|-|runtime|
|test|test|-|-|test|
|provide|provide|-|provide|provide|
|runtime|runtime|-|-|runtime|
上表所示，最左边一列表示第一直接依赖范围，最上面一行表示第二直接依赖范围，中间的交叉单元格则表示传递性依赖范围

## 依赖的调解

当传递性依赖造成问题的时候，我们就需要清楚的知道该传递性依赖是从哪条路径引入的。

例如： 项目A有这样的依赖关系: A->B->C->X(1.0)、A->D->X(2.0), X是A的传递性依赖，但是两条依赖路径上有两个版本的X,那么哪个X会被Maven解析使用呢? 

Maven 依赖调解( Dependency Mediation)的

> 第一原则是:路径最近者优先。

该例中X(1.0)的路径长度为3，而X(2.0)的路径长度为2，因此X(2.0)会被解析使用。

> 第二原则： 依赖声明的顺序在前的优先

比如这样的依赖关系: A->B->Y(1.0)、 A->C-> Y(2.0)，Y(1. 0)和Y(2. 0)的依赖路径长度是一样的，都为2。Maven 定义了依赖调解的第二原则:第一声明者优先。在依赖路径长度相等的前提下，在POM中依赖声明的顺序决定了谁会被解析使用，顺序最靠前的那个依赖优胜。该例中，如果B的依赖声明在C之前，那么Y (1.0)就会被解析使用。

## 可选依赖

假设有这样一个依赖关系，项目A依赖于项目B,项目B依赖于项目X和Y, B对于X和Y的依赖都是可选依赖:A->B、B->X(可选)、B->Y(可选)。根据传递性依赖的定义，如果所有这三个依赖的范围都是compile,那么X、Y就是A的compile范围传递性依赖。然而，由于这里X、Y是可选依赖，依赖将不会得以传递。换句话说，X、Y将不会对A有任何影响

> 1、显示声明：如果A需要使用B所依赖的X或者Y，只能在A中显示的声明该依赖。

> 2、尽量避免使用：关于可选依赖需要说明的一点是，在理想的情况下，是不应该使用可选依赖的。前面我们可以看到，使用可选依赖的原因是某一个项目实现了多个特性，在面向对象设计中，有个**单一职责性原则**，意指一个类应该只有一项职责，而不是糅合太多的功能。这个原则在规划Maven项目的时候也同样适用。

# 依赖的最佳实践
## 排除依赖
当项目A依赖于项目B，同时不想引入传递依赖C，就可以使用exclusions排除依赖C，同时在A中显示声明C。
``` xml
<project>
	<groupId>com.xxx.xx</groupId>
	<artifactId>project-a</artifactId>
	<version>1.O.O</version>
	<dependencies>
		<dependency>
			<groupId>com.xxx.xx</groupId>
			<artifactId>project-b</artifactId>
			<version>1.0.0</version>
			<exclusions>
				<exclusion>
					<groupId>com.xxx.xx</groupId>
					<artifactId>project-c</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.xxx.xx</groupId>
			<artifactId>project-c</artifactId>
			<version>1.1.0</version>
		</dependency>
	</dependencies>
</project>
```
代码中使用exclusions 元素声明排除依赖，exclusions 可以包含一个或者多个exclusion子元素，因此可以排除一个或者多个传递性依赖。需要注意的是，声明exclusion 的时候只需要groupId 和artifactld, 而不需要version元素，这是因为只需要groupId和artifactId就能唯一定位依赖图中的某个依赖。

换句话说，Maven 解析后的依赖中，不可能出现groupId和artifactld 相同，但是version不同的两个依赖。

## 归类依赖
统一定义版本号，进行版本控制。示例如下：
``` xml
<properties>
    <fastjson.version>1.2.46</fastjson.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Edgware.SR3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 优化依赖

> 1、mvn dependency:list

查看当前项目已解析的依赖

> 2、mvn dependency:tree

查看当前项目的依赖树

> 3、mvn dependency:analyze

分析当前项目的依赖

结果中重要的是两个部分。、
1. 首先是Used undeclared dependencies,意指项目中使用到的，但是没有显式声明的依赖。

显式声明任何项目中直接用到的依赖。

2. 还有一个重要的部分是Unused declared dependencies.意指项目中未使用的，但显式声明的依赖，这里有spring-core 和spring-beans.需要注意的是，对于这样一类依赖，我们不应该简单地直接删除其声明，而是应该仔细分析。

由于dependency:analyze只会分析编译主代码和测试代码需要用到的依赖，一些执行测试和运行时需要的依赖它就发现不了


