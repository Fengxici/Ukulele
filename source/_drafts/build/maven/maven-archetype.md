---
title: Maven Archetype
date: 2019-09-06 17:04:15
tags:
- Maven
- Archetype
categories:
- 构建
- Maven
---
Maven 使用 archetype(原型) 来创建自定义的项目结构，形成 Maven 项目模板。

archetype是一个 Maven 插件，准确说是一个项目模板，它的任务是根据模板创建一个项目结构。我们将使用 quickstart 原型插件创建一个简单的 java 应用程序。

使用项目模板
让我们打开命令控制台，执行以下 mvn 命令(准备好一个目录，在该目录下打开命令行窗口):

> mvn archetype:generate 

Maven 将开始处理，并要求选择所需的原型:
有很多类似这样的输出 
> 600: remote -> org.trailsframework:trails-archetype (-)

最后一行输出
> Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 1659:

注：你的可能不是1659

选择你想要使用的模板的编号，如600，回车

Maven 将询问原型的版本，类似这样
```
Choose org.apache.maven.archetypes:maven-archetype-quickstart version:
1: 1.0-alpha-1
2: 1.0-alpha-2
3: 1.0-alpha-3
4: 1.0-alpha-4
5: 1.0
6: 1.1
Choose a number: 6:
```

按下 Enter 选择默认选项(或者先输入一个版本编号)

Maven 将询问项目细节。按要求输入项目细节。如果要使用默认值则直接按 Enter 键。你也可以输入自己的值。
```
Define value for property 'groupId': : 
Define value for property 'artifactId': : 
Define value for property 'version': 
Define value for property 'package': 
```

Maven 将要求确认项目细节，按 Enter 或按 Y

现在 Maven 将开始创建项目结构

完成后项目将创建完成