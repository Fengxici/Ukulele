---
title: 第二章 注册中心Eureka
date: 2019-08-30 09:19:00
tags: 
- spring cloud
- eureka
categories:
- spring cloud
- 教程
---

> 微服务架构中的服务治理

官方指导书：https://spring.io/projects/spring-cloud#learn

本章主要带大家一起体验Spring Cloud微服务架构中的一个微服务治理模块——Eureka。通过本章的学习您将获得的技能有：
> - 搭建服务注册中心
> - 服务的注册
> - 注册中心集群

# 一些啰嗦
## 注册中心的作用
注册中心是微服务架构中最为核心和基础的模块，它主要用来实现各个微服务实例的自动化注册和发现。
## spring cloud支持哪些注册中心
Spring Cloud的注册中心实现其实存在多种方式，比如 Hashicorp Consul 或 Apache Zookeeper。本章我们只介绍Eureka。Eureka分为服务端和客户端。服务端即我们所说的注册中心，它支持高可用的集群配置；客户端主要处理服务的注册和发现。客户端嵌入在提供各种服务的应用中，在应用程序运行时向注册中心注册自身提供的服务并周期性地发送心跳来更新它的服务租约。同时，它也从服务端查询当前注册的服务信息并把他们缓存到本地并周期性地刷新服务状态。

您若有兴趣进一步了解可以访问文章开头提供官方文档地址进行查阅。

# 动手实践
纸上得来终觉浅，绝知此事要躬行。接下来我们动手体验一下(IDEA)。
## 1 创建父项目eureka-demo

创建好项目后删除项目下的src目录，这里将erueka-demo做为一个maven父项目，所以不需要源码目录。只保留目录下的 **.idea**文件夹、**.iml**文件(可能没有)和**pom.xml**文件。

## 2 修改pom.xml，引入依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>timing.springcloud</groupId>
    <artifactId>eureka-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>eureka-demo</name>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
    </parent>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <profiles>
        <profile>
            <id>spring</id>
            <repositories>
                <repository>
                    <id>spring-snapshots</id>
                    <name>Spring Snapshots</name>
                    <url>https://repo.spring.io/libs-snapshot-local</url>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                </repository>
                <repository>
                    <id>spring-milestones</id>
                    <name>Spring Milestones</name>
                    <url>https://repo.spring.io/libs-milestone-local</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>spring-releases</id>
                    <name>Spring Releases</name>
                    <url>https://repo.spring.io/release</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>spring-snapshots</id>
                    <name>Spring Snapshots</name>
                    <url>https://repo.spring.io/libs-snapshot-local</url>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                </pluginRepository>
                <pluginRepository>
                    <id>spring-milestones</id>
                    <name>Spring Milestones</name>
                    <url>https://repo.spring.io/libs-milestone-local</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>spring-releases</id>
                    <name>Spring Releases</name>
                    <url>https://repo.spring.io/libs-release-local</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
        <profile>
            <id>java9+</id>
            <activation>
                <jdk>[9,)</jdk>
            </activation>
            <dependencies>
                <dependency>
                    <groupId>javax.activation</groupId>
                    <artifactId>javax.activation-api</artifactId>
                </dependency>
                <dependency>
                    <groupId>javax.xml.bind</groupId>
                    <artifactId>jaxb-api</artifactId>
                </dependency>
                <dependency>
                    <groupId>org.glassfish.jaxb</groupId>
                    <artifactId>jaxb-runtime</artifactId>
                    <optional>true</optional>
                </dependency>
            </dependencies>
        </profile>
    </profiles>
</project>
```
有两点需要注意：打包类型应为pom。

至此，父项目创建完毕。

## 3 创建注册中心服务端eureka-server
新建maven module， 命名为eureka-server.此时父项目里的pom.xml多出了如下内容
``` xml
<modules>
    <module>eureka-server</module>
</modules>
```
module创建成功，接下来做一点简单的调整
> 1 main路径下添加resources文件夹，
> 2 添加报名eureka.server，
> 3 删除不需要的代码AppTest类和App类
> 4 erueka-server module上右键open module settings将新建的resources文件夹标记为Resources。

## 4 修改eureka-server的pom，引入eureka的服务器端依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>eureka-demo</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>eureka-server</artifactId>
    <name>eureka-server</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

引入的依赖包括spring-cloud-starter-netflix-eureka-server和spring-boot-starter-actuator。应为注册中心也是一个Spring Boot项目，所以引入actuator以备后期运维和监控使用。这里的spring-cloud-starter-netflix-eureka-server即是实现注册中心最核心的依赖。

## 5 添加启动类，实现注册中心
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

这跟普通的Spring Boot启动类的唯一区别就是在类名前面多了一个**@EnableEurekaServer**注解。正是这句注解让我们很轻松的就实现了一个注册中心的创建。

reimport一下**eureka-server的pom**，待出现EurekaServerApplication的运行配置即可点击运行或者调试按钮将注册中心跑起来了。

此时一个注册中心就创建成功了。可以打开浏览器（http://localhost:8080/eureka)查看效果

细心的您肯定看到了控制台一直在报错。这是因为在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为。resources文件夹里添加application.yml文件，并设置如下内容：
``` yml
eureka:
    client:
#默认情况下，应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
        register-withEureka: false
#表示不去检索其他的Eureka Server获取注册信息，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
        fetch-registry: false
        service-url:
# 对外暴露的地址
            defaultZone: http://localhost:8080/eureka/
```
此时重启应用，已经没有报错信息了。

（还记得第一章演示的热部署吗，动手加进来试试吧）

## 6 安全设置

您也许已经注意到了，我们打开浏览器访问8080端口就可以直接看到注册中心的相关信息了，没有任何阻拦，这似乎很不合理。同样，如果我们此时设置打开actuator敏感信息的端点也会被所有人看到。的确，生产环境中，这样的情况是不允许的。对所有的请求进行一个身份认证的拦截，似乎是一个不错的方案。
``` xml
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    ...
</dependencies>
```
spring-boot-starter-security正式这样一个组件。其核心是spring-security-web。

引入依赖之后，重新启动应用。再次访问8080端口的时候就跳转到了登陆界面.

那么用户名和密码是什么呢。

请仔细翻看您的控制台，您会看到一串字符串
> Using generated Security password:xxxxxxxxx

这即是密码！密码是随机生成的，您的控制台肯定与此不同。用户名默认为user。在登陆界面输入用户名和密码即可进入注册中心界面。

是的，您也许已经想到了，我们可不可以自己设置用户名和密码呢？当然可以！

同样，在application.yml中添加如下设置
``` yml
spring:
    security:
        user:
            name: root
            password: root
```
并且将
``` yml
defaultZone: http://localhost:8080/eureka/
```
修改为
``` yml
defaultZone: http://root:root@localhost:8080/eureka/
```
这里我将用户名和密码全部设置成root，您可以设置自己的用户名和密码。此时再次访问8080端口，即可使用自己设置的用户名和密码登陆了。

注册中心创建好了。接下来我们动手创建客户端吧。

## 7 创建Eurekak客户端项目eureka-client
与第3步类似，创建一个maven module。命名为eureka-client。同样，我们也需要对项目结构做一下调整。
父项目eureka-demo的pom.xml里的modules节点变成了这样：
``` xml
<modules>
    <module>eureka-server</module>
    <module>eureka-client</module>
</modules>
```
## 8 修改eureka-client的pom，引入eureka的客户端依赖
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>eureka-demo</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>eureka-client</artifactId>
    <name>eureka-client</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
**spring-cloud-starter-netflix-eureka-client**即是erueka客户端的关键组件.

## 9 添加启动类
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class EurekaClientApplication {
    public static void main(String[] args){
        SpringApplication.run(EurekaClientApplication.class,args);
    }
}
```
此时还不可以启动应用，我们需要配置一下，告知客户端注册中心在哪里。

resources目录下新增application.yml文件。并添加配置如下
``` yml
spring:
    application:
        name: eureka-client
server:
    port: 8081
eureka:
    client:
        service-url:
            defaultZone: http://root:root@localhost:8080/eureka/
```
由于默认的8080端口已被服务端使用，我这里改用8081端口，并且给应用设置一个名字，便于在注册中心中识别。

到此客户端就创建成功了。

但是，此时如果向服务端注册会报注册不成功错误。这是因为我们引入了security，需要把服务端的CSRF保护去掉。我们在eureka-server项目里添加配置类如下：
``` java
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.autoconfigure.security.SecurityProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
@ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
public class SpringBootWebSecurityConfiguration {
    @Configuration
    @Order(SecurityProperties.BASIC_AUTH_ORDER)
    static class DefaultConfigurerAdapter extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            super.configure(http);
            http.csrf().disable();
        }
    }
}
```
到此，完成的注册中心服务端和客户端就创建好了。
将eureka-server和eureka-client分别运行起来，浏览器访问8080端口，输入用户名和密码，即可观察到效果。

注册中心是如此的重要！如果宕机，整套系统就瘫痪了。好在eureka与其他注册中心一样，可以集群化部署。接下来，我们动手改造下eureka-server项目以实现注册中心的集群化部署。

## 10 集群化部署.

要做到集群化部署，只需修改配置文件即可。修改**eureka-server**的application.yml如下：
``` yml
# 安全认证的配置
#spring:
#  security:
#    user:
#      name: root
#      password: root
#eureka:
#  client:
#        #默认情况下，应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
#    register-withEureka: false
#        #表示不去检索其他的Eureka Server获取注册信息，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
#    fetch-registry: false
#    service-url:
##        对外暴露的地址
#      defaultZone: http://root:root@localhost:8080/eureka/

# ============================以上为单机部署配置========================================

#=============================以下为集群部署配置========================================

spring:
  application:
    name: eureka-server
# 安全认证的配置
  security:
    user:
      name: root
      password: root
  profiles:
    active: peer1

---
server:
  port: 8000
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
        #默认情况下，应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
    register-with-eureka: false
        #表示不去检索其他的Eureka Server获取注册信息，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
    fetch-registry: false
    service-url:
        #对外暴露的地址
      defaultZone: http://root:root@localhost:8001/eureka/,http://root:root@hosthost:8002/eureka/


---
server:
  port: 8001
spring:
  profiles: peer2
   # 安全认证的配置
eureka:
  instance:
    hostname: peer2
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
        #默认情况下，应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
    register-with-eureka: false
        #表示不去检索其他的Eureka Server获取注册信息，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
    fetch-registry: false
    service-url:
        #对外暴露的地址
      defaultZone: http://root:root@localhost:8000/eureka/,http://root:root@localhost:8002/eureka/

---
server:
  port: 8002
spring:
  profiles: peer3
eureka:
  instance:
    hostname: peer3
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
        #默认情况下，应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
    register-with-eureka: false
#        表示不去检索其他的Eureka Server获取注册信息，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
    fetch-registry: false
    service-url:
        #对外暴露的地址
      defaultZone: http://root:root@localhost:8000/eureka/,http://root:root@localhost:8001/eureka/
```
将单机版的配置注释掉，新增3个profile，分别使用8001，8002，8003三个端口。Eureka服务端的高可用实际上就是将自己作为服务向其他注册中心注册自己，这样就可以形成一组互相注册的服务注册中心，以实现服务清单的互相同步，达到高可用的效果。

修改**eureka-client**的application.yml文件：
``` yml
spring:
  application:
    name: eureka-client

server:
  port: 8081
eureka:
  client:
    service-url:
#    单机
#      defaultZone: http://root:root@localhost:8000/eureka/
#集群
      defaultZone: http://root:root@localhost:8000/eureka/,http://root:root@localhost:8001/eureka/,http://root:root@localhost:8002/eureka/
```
## 11 集群效果观察

打开eureka-server文件夹，

> 1 shift+右键打开powershell或cmd窗口，输入mvn clean package命令将eureka-server应用打包，
> 2 打包好后cd进入target目录，输入java -jar eureka-server-1.0-SNAPSHOT.jar将应用运行起来，此时可以看到注册中心使用的是8000端口
> 3 进入target目录，shift+右键打开powershell或cmd窗口，输入java -jar eureka-server-1.0-SNAPSHOT.jar --spring.profiles.active=peer2,运行起来第二个注册中心应用，此时可以看到这次打开的注册中心使用的是8001端口
> 4 进入target目录，shift+右键打开powershell或cmd窗口，输入java -jar eureka-server-1.0-SNAPSHOT.jar --spring.profiles.active=peer3,运行起来第三个注册中心应用，此时可以看到这次打开的注册中心使用的是8002端口
> 5 回到IEDA或者和上面的方式类似，运行起客户端应用eureka-client.

此时打开浏览器依次访问三个注册中心，观察效果。

客户端向第一个和第二个注册中心注册了，如果关闭第一个，客户端又会向第二个第三个注册。

这样，注册中心的高可用得到了很好的保障。

至此，本章关于注册中心的介绍就全部结束了。希望勤于动手和思考的您已经有所收获。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。