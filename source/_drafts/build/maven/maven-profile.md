---
title: Maven 构建配置文件
date: 2019-09-06 17:04:15
tags:
- Maven
- profile
categories:
- 构建
- Maven
---

构建配置文件是一系列的配置项的值，可以用来设置或者覆盖 Maven 构建默认值。

使用构建配置文件，你可以为不同的环境，比如说生产环境（Production）和开发（Development）环境，定制构建方式。

配置文件在 pom.xml 文件中使用 activeProfiles 或者 profiles 元素指定，并且可以通过各种方式触发。配置文件在构建时修改 POM，并且用来给参数设定不同的目标环境，比如说，开发（Development）、测试（Testing）和生产环境（Producation）中数据库服务器的地址。

# 构建配置文件的类型
- 项目级（Per Project） 定义在项目的POM文件pom.xml中
- 用户级 （Per User） 定义在Maven的设置xml文件中 (%USER_HOME%/.m2/settings.xml)
- 全局（Global）	定义在 Maven 全局的设置 xml 文件中 (%M2_HOME%/conf/settings.xml))

# 配置文件激活
Maven的构建配置文件可以通过多种方式激活。
- 使用命令控制台输入显式激活。
- 通过 maven 设置。
- 基于环境变量（用户或者系统变量）。
- 操作系统设置（比如说，Windows系列）。
- 文件的存在或者缺失。

## 1、配置文件激活
profile 可以让我们定义一系列的配置信息，然后指定其激活条件。这样我们就可以定义多个 profile，然后每个 profile 对应不同的激活条件和配置信息，从而达到不同环境使用不同配置信息的效果。
``` xml
 <profiles>
    <profile>
        <id>test</id>
        <build>
           ......
        </build>
    </profile>
    <profile>
        <id>normal</id>
        <build>
            ......
        </build>
    </profile>
    <profile>
        <id>prod</id>
        <build>
            ......
        </build>
    </profile>
</profiles>
```
注意：构建配置文件采用的是 profiles 节点。

说明：上面新建了三个 profiles，其中 <id> 区分了不同的 profiles 执行不同的任务；通过命令行参数输入指定的id来指定哪个profile生效。

执行命令：
> mvn test -Ptest

提示：第一个 test 为 Maven 生命周期阶段，第 2 个 test 为构建配置文件指定的 id 参数，这个参数通过 -P 来传输，当然，它可以是 prod 或者 normal 这些由你定义的id。

## 2、通过Maven设置激活配置文件
打开maven根目录下的 config/settings.xml 文件。配置 setting.xml 文件，增加 activeProfiles属性：
``` xml
<settings xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
   ...
   <activeProfiles>
      <activeProfile>test</activeProfile>
   </activeProfiles>
</settings>
```
执行命令
> mvn test

此时不需要使用 -Ptest 来输入参数了，上面的 setting.xml 文件的 <activeprofile> 已经指定了 test 参数代替了。

## 3、通过环境变量激活配置文件
``` xml
 <profiles>
      <profile>
          <id>test</id>
          <activation>
            <property>
               <name>env</name>
               <value>test</value>
            </property>
          </activation>
          <build>
            ......
          </build>
      </profile>
      <profile>
          <id>normal</id>
          <build>
              ......
          </build>
      </profile>
      <profile>
          <id>prod</id>
          <build>
              ......
          </build>
      </profile>
   </profiles>
```
执行命令：
> mvn test -Denv=test

上面使用 -D 传递环境变量，其中 evn 对应刚才设置的 name 值，test 对应value。

**注**：先把上一步测试的 setting.xml 值全部去掉。

## 4、通过操作系统激活配置文件
``` xml
<profile>
   <id>test</id>
   <activation>
      <os>
         <name>Windows XP</name>
         <family>Windows</family>
         <arch>x86</arch>
         <version>5.1.2600</version>
      </os>
   </activation>
</profile>
```
activation 元素包含操作系统信息。当系统为 windows XP 时，test Profile 将会被触发。

现在打开命令控制台，跳转到 pom.xml 所在目录，并执行下面的 mvn 命令。不要使用 -P 选项指定 Profile 的名称。Maven 将显示被激活的 test Profile 的结果。
> mvn test

## 5、通过文件的存在或者缺失激活配置文件
``` xml
<profile>
   <id>test</id>
   <activation>
      <file>
         <missing>target/generated-sources/a.txt</missing>
      </file>
   </activation>
</profile>
```
使用 activation 元素包含文件信息。当 target/generated-sources/a.txt 缺失时，test Profile 将会被触发。
现在打开命令控制台，跳转到 pom.xml 所在目录，并执行下面的 mvn 命令。不要使用 -P 选项指定 Profile 的名称。Maven 将显示被激活的 test Profile 的结果。
> mvn test