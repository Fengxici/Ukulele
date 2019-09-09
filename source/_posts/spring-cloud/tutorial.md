---
title: 综合案例
date: 2019-09-08 14:21:46
tags:
- spring cloud
- 综合案例
categories:
- spring cloud
- 教程
---

经过前十一章的动手训练，相信读者朋友们已经能够相对熟练的创建spring cloud各个组件的项目了。本篇我们将前面的所学联系在一起，创建一个综合演练项目，看看个组件联系在一起是如何工作的。
本篇源码对应位置：[github](https://github.com/Fengxici/spring-cloud-current-tutorial)

**依然建议读者先自行动手创建项目，完成之后再和源码做对照**

这次我们先编排和各个服务的端口，并按照如下顺序创建项目

1. 注册中心，集群化，部署三套，端口分别为8000，8001，8002
2. 配置中心，配置信息放在本地文件系统，端口为8888，注册至注册中心
3. 服务监控，端口使用13000，注册至注册中心
4. zipkin,端口使用7000，注册至注册中心
5. turbine，端口使用11000，注册至注册中心
6. gateway,端口使用12000，注册至注册中心
7. 生产者服务，端口使用9000，注册至注册中心
8. 消费者服务，端口使用14000，注册至注册中心

---
# 父项目
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>timing.springcloud</groupId>
    <artifactId>tutorial-master</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>tutorial-master</name>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
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
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
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

# 注册中心
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-register</artifactId>
    <name>tutorial-register</name>
    <description>注册中心</description>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
## 配置
resources/application.yml
``` yml
spring:
  application:
    name: registerServer
# 安全认证的配置
  security:
    user:
      name: root
      password: root
  profiles:
    active: peer1
management:
  endpoints:
    web:
      exposure:
        include: "*"
#        SpringBootActuator监控暴露所有接口
  endpoint:
     health:
          show-details: ALWAYS

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
      defaultZone: http://root:root@localhost:8001/eureka/,http://root:root@localhost:8002/eureka/

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
        #表示不去检索其他的Eureka Server获取注册信息，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
    fetch-registry: false
    service-url:
        #对外暴露的地址
      defaultZone: http://root:root@localhost:8000/eureka/,http://root:root@localhost:8001/eureka/
```
## 启动类
``` java
package timing.springcloud.tutorial.register;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class RegisterApplication {
    public static void main(String[] args){
        SpringApplication.run(RegisterApplication.class,args);
    }
}
```
## 安全
``` java
package timing.springcloud.tutorial.register;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication.Type;
import org.springframework.boot.autoconfigure.security.SecurityProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
@ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
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

# 配置中心
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-config-native</artifactId>
    <name>tutorial-config-native</name>
    <description>配置中心-本地版</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
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

## 配置
resources/application.yml
``` yml
server:
  port: 8888
spring:
  application:
    name: config-local-service
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          searchLocations: classpath:/config
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka
```
## 启动类
``` java
package timing.springcloud.config.local;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;
@SpringBootApplication
@EnableConfigServer
public class ConfigLocalApplication {
    public static void main(String[] args){
        SpringApplication.run(ConfigLocalApplication.class,args);
    }
}
```
## 配置仓库
resources/conf/conf-dev.yml
``` yml
provider:
  hello: hello, i am provider
consumer:
  hello: hello, i am consumer
```
注：此处文件名称要存放位置不能修改，与其他配置保持一直。如果你不太理解，请与示例保持一直，后续会有详细介绍。

# 监控服务
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-monitor</artifactId>
    <name>tutorial-monitor</name>
    <description>监控服务</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
            <version>2.1.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
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
## 配置
``` yml
server:
  port: 13000

spring:
  application:
    name: monitor-service
  security:
    user:
      name: root
      password: root

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka
```
## 启动类
``` java
package timing.springcloud.tutorial.monitor;
import de.codecentric.boot.admin.server.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@EnableAdminServer
@SpringBootApplication
public class MonitorApplication {
    public static void main(String[] args) {
        SpringApplication.run(MonitorApplication.class, args);
    }
}
```
# Zipkin
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-sleuth-zipkin</artifactId>
    <name>tutorial-sleuth-zipkin</name>
    <description>链路追踪</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
            <version>2.12.9</version>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <scope>runtime</scope>
            <version>2.12.9</version>
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
## 配置
``` yml
QUERY_PORT: 7000
spring:
  application:
    name: sleuth-zipkin-server
  security:
      user:
        name: root
        password: root
  profiles:
    include: shared

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${QUERY_PORT:9411}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka

management:
  endpoints:
    web:
      exposure:
        include: "*"
#        SpringBootActuator监控暴露所有接口
  endpoint:
     health:
          show-details: ALWAYS
```
## 启动类
``` java
package timing.springcloud.sleuth.zipkin;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import zipkin2.server.internal.EnableZipkinServer;
import zipkin2.server.internal.RegisterZipkinHealthIndicators;
@SpringBootApplication
@EnableZipkinServer
public class SleuthZipkinApplication {
    public static void main(String[] args){
        new SpringApplicationBuilder(SleuthZipkinApplication.class)
                .listeners(new RegisterZipkinHealthIndicators())
                .properties("spring.config.name=zipkin-server")
                .run(args);
    }
}
```
# Turbine
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-turbine</artifactId>
    <name>tutorial-turbine</name>
    <description>聚合Hystrix</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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
## 配置
``` yml
server:
  port: 11000

spring:
  application:
    name: turbine-dashboard
  security:
    user:
      name: root
      password: root


eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka

turbine:
  appConfig: consumer-service
  aggregator:
    clusterConfig: default
  clusterNameExpression: new String("default")

management:
  endpoints:
    web:
      exposure:
        include: "*"
#        SpringBootActuator监控暴露所有接口
  endpoint:
     health:
          show-details: ALWAYS
```
## 启动类
``` java
package timing.springcloud.tutorial.turbine;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.cloud.netflix.turbine.EnableTurbine;
@SpringBootApplication
@EnableHystrixDashboard
@EnableTurbine
public class TurbineDashboardApplication {
    public static void main(String[] args){
        SpringApplication.run(TurbineDashboardApplication.class,args);
    }
}
```
# Gateway
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-gateway</artifactId>
    <name>tutorial-gateway</name>
    <description>zuul网关</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
        <!--添加链路追踪支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
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
## 配置
``` yml
server:
  port: 12000

spring:
  application:
    name: gateway
  zipkin:
    base-url: http://localhost:7000
  sleuth:
    sampler:
      probability: 1.0

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka

zuul:
  routes:
    provider:
      path: /api/**
      serviceId: consumer-service

management:
  endpoints:
    web:
      exposure:
        include: "*"
  #        SpringBootActuator监控暴露所有接口
  endpoint:
    health:
      show-details: ALWAYS
```
## 启动类
``` java
package timing.springcloud.tutorial.gateway;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
@SpringBootApplication
@EnableZuulProxy
public class GateWayApplication {
    public static void main(String[] args){
        SpringApplication.run(GateWayApplication.class,args);
    }
}
```
# 生产者服务
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-provider</artifactId>
    <name>tutorial-provider</name>
    <description>服务提供者</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- 从配置服务器读取配置 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--添加链路追踪支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
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
## 配置
``` yml
server:
  port: 9000
spring:
  application:
    name: provider-service
  security:
    user:
      name: root
      password: root
  zipkin:
      base-url: http://localhost:7000
  sleuth:
    sampler:
      probability: 1.0
  cloud:
    config:
      name: config
      profile: dev
      uri: http://localhost:8888/
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
    service-url:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka

management:
  endpoints:
    web:
      exposure:
        include: "*"
#        SpringBootActuator监控暴露所有接口
  endpoint:
    health:
      show-details: ALWAYS
```
## 启动类
``` java
package timing.springcloud.tutorial.provider;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```
## contoller
``` java
package timing.springcloud.tutorial.provider.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletResponse;

@RestController
@RefreshScope
public class HelloController {

    @Value("${provider.hello}")
    private String hello;

    @RequestMapping("/hello")
    public String index(@RequestParam String name) {
        return "hello "+name+"，this is first messge";
    }

    @Value("${spring.application.name}")
    private String name;

    @GetMapping("/service")
    @ResponseBody
    public String test(HttpServletResponse response, String para) {
        return para + "_service";
    }

    @GetMapping("/feignservice/{para}")
    @ResponseBody
    public String feignservice(HttpServletResponse response, @PathVariable String para) {
        return para + "_service";
    }



    @RequestMapping("/provider")
    public String from() {
        return this.hello;
    }
}
```
# 消费者服务
## pom
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tutorial-master</artifactId>
        <groupId>timing.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>tutorial-consumer</artifactId>
    <name>tutorial-consumer</name>
    <description>服务消费者</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 从配置服务器读取配置 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <!--增加feign服务代理依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--增加熔断依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <!--添加链路追踪支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
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
## 配置
``` yml
server:
  port: 14000

service:
  url: http://consumer-service

spring:
  application:
    name: consumer-service
  security:
    user:
      name: root
      password: root
  zipkin:
    base-url: http://localhost:7000
  sleuth:
    sampler:
      probability: 1.0
  cloud:
    config:
      name: config
      profile: dev
      uri: http://localhost:8888/
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
    instance-id: ${eureka.instance.hostname}:${server.port}
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:8000/eureka,http://root:root@localhost:8001/eureka,http://root:root@localhost:8002/eureka

management:
  endpoints:
    web:
      exposure:
        include: "*"
  #        SpringBootActuator监控暴露所有接口
  endpoint:
    health:
      show-details: ALWAYS
```
## 启动类
``` java
package timing.springcloud.tutorial.consumer;
import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
@SpringBootApplication
@EnableFeignClients
@EnableCircuitBreaker
@EnableHystrixDashboard
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
    @LoadBalanced //开启负载均衡客户端
    @Bean //注册一个具有容错功能的RestTemplate
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
    // 这个是2.02要添加的，不然仪表盘不显示
    @Bean
    public ServletRegistrationBean getServlet(){
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean<>(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```
## 其他
``` java
package timing.springcloud.tutorial.consumer.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.servlet.ModelAndView;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
@RestController
@RefreshScope
public class ClientController {
	@Value("${service.url}")
	private String serviceUrl;
	@Autowired
	private RestTemplate restTemplate;
	@Autowired
    private DiscoveryClient client;
	@Value("${consumer.hello}")
	private String hello;
	@RequestMapping("/consumer")
	public String from() {
		return this.hello;
	}
	@GetMapping("/index")
	public ModelAndView index() {
		test();
		ModelAndView modelAndView = new ModelAndView("/index.html");
		return modelAndView;
	}
	@GetMapping("/service")
	public Map<String,String> service() {
		Map<String,String> map = new HashMap<>();
		map.put("mykey", "mydata");
		return map;
	}
	private void test() {
		List<ServiceInstance> instances = client.getInstances("provider-service");
        for (int i = 0; i < instances.size(); i++) {
        	System.out.println("/hello,host:" + instances.get(i).getHost() + ",service_id:" + instances.get(i).getServiceId());
        }
        System.out.println(serviceUrl);
		String ret = restTemplate.getForObject(serviceUrl+"/service", String.class);
		System.out.println("response  = " + ret);

		ret = restTemplate.getForObject(serviceUrl+"/service?para=test", String.class);
		System.out.println("response1  = " + ret);
	}
}

package timing.springcloud.tutorial.consumer.controller;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import timing.springcloud.tutorial.consumer.feign.MyFeignClient;
@RestController
public class FeignController {
    @Autowired
    private MyFeignClient feignClient;
    @GetMapping(value = "/testFeign/{para}")
    @HystrixCommand(fallbackMethod = "fallback")
    public String testFeign(@PathVariable("para") String para) {
        String ret = feignClient.testFeign(para);
        System.out.println("feignresponse=" + ret);
        return ret;
    }
    @GetMapping(value = "/testHystrix/{para}")
    @HystrixCommand(fallbackMethod = "fallback")
    public String testHystrix(@PathVariable("para") String para) {
        String ret = feignClient.testHystrix(para);
        System.out.println("feignresponse=" + ret);
        return ret;
    }
    public String fallback(String para) {
        String temp = "testHystrix调用失败！参数为：" + para;
        System.out.println(temp);
        return temp;
    }
}

package timing.springcloud.tutorial.consumer.feign;
import feign.auth.BasicAuthRequestInterceptor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BasicAuthConfiguration {
	@Value("${spring.security.user.name}")
	private String username;
	@Value("${spring.security.user.password}")
	private String password;
    @Bean
    public BasicAuthRequestInterceptor basicAuthorizationInterceptor() {
        return new BasicAuthRequestInterceptor(username, password);
    }
}

package timing.springcloud.tutorial.consumer.feign;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;
@Component
public class HystrixClientFallbackFactory implements FallbackFactory<MyFeignClient> {
	@Override
	public MyFeignClient create(Throwable cause) {
		return new MyFeignClient() {
			@Override
			public String testFeign(String para) {
				cause.printStackTrace();
				return cause.getMessage();
			}
			@Override
			public String testHystrix(String para) {
				cause.printStackTrace();
				return cause.getMessage();
			}
		};
	}
}

package timing.springcloud.tutorial.consumer.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
@FeignClient(name = "provider-service", configuration = BasicAuthConfiguration.class,  fallbackFactory = HystrixClientFallbackFactory.class)
public interface MyFeignClient {
	/**
	 * 隐射到rest-service
	 * @param para
	 * @return
	 */
    @GetMapping(value = "/feignservice/{para}")
    String testFeign(@PathVariable("para") String para);
    @GetMapping(value = "/testHystrix/{para}")
    String testHystrix(@PathVariable("para") String para);
}
```

至此，所有的服务创建完毕。接下来挨个跑起来吧。

# 启动
首先我们在项目根目录打开命令行窗口，执行一下命令
> mvn package

将所有的模块都打包成可运行的**.jar**文件。接下来将每个项目target目录下的.jar文件拷贝到同一个文件夹下，备用。

现在我们在jar包存放目录下一次启动每个服务。请注意启动顺序，这里注册中心必须先启动，配置中心必须在消费者服务和提供者服务之前启动

## 注册中心
这里启动三个注册中心，分别打开三个命令行窗口（当前目录下），并分别运行一下命令
>java -jar .\tutorial-register-1.0-SNAPSHOT.jar

>java -jar .\tutorial-register-1.0-SNAPSHOT.jar --spring.profiles.active=peer2

>java -jar .\tutorial-register-1.0-SNAPSHOT.jar --spring.profiles.active=peer3

注：第一条命令也可以是
>ja va -jar .\tutorial-register-1.0-SNAPSHOT.jar --spring.profiles.active=peer1

浏览器分别打开
- http://localhost:8000/eureka
- http://localhost:8001/eureka
- http://localhost:8002/eureka

可以看到如下三个界面，注册中心启动完成

![eureka-8000](/images/spring-cloud/tutorial/eureka-8000.jpg)

![eureka-8001](/images/spring-cloud/tutorial/eureka-8001.png)

![eureka-8002](/images/spring-cloud/tutorial/eureka-8002.png)

## 配置中心
打开命令行窗口，运行
>java -jar .\tutorial-config-native-1.0-SNAPSHOT.jar

打开浏览器访问
· http://localhost:8888/config/dev

![config](/images/spring-cloud/tutorial/config.png)

注册中心启动完成

## 监控服务
打开命令行窗口，运行
>java -jar .\tutorial-monitor-1.0-SNAPSHOT.jar

打开浏览器访问
- http://localhost:13000/#/wallboard

![monitor](/images/spring-cloud/tutorial/monitor.jpg)

监控服务启动完成

## Zipkin
打开命令行窗口，运行
>java -jar .\tutorial-sleuth-zipkin-1.0-SNAPSHOT.jar

打开浏览器访问

链路跟踪服务启动完成
- http://localhost:7000

![zipkin](/images/spring-cloud/tutorial/zipkin.png)

## Turbine
打开命令行窗口，运行
>java -jar .\tutorial-turbine-1.0-SNAPSHOT.jar

打开浏览器访问
- http://localhost:11000/hystrix

![turbine](/images/spring-cloud/tutorial/turbine.png)

熔断监控服务启动完成

## 网关服务
打开命令行窗口，运行
>java -jar .\tutorial-gateway-1.0-SNAPSHOT.jar

## 提供者服务
这里我们启动两个提供者服务，打开命令行窗口，运行
>java -jar .\tutorial-provider-1.0-SNAPSHOT.jar
>java -jar .\tutorial-provider-1.0-SNAPSHOT.jar -server.port=15000

## 消费者服务
这里我们打开两个消费者服务，打开命令行窗口，运行
>java -jar .\tutorial-consumer-1.0-SNAPSHOT.jar
>java -jar .\tutorial-consumer-1.0-SNAPSHOT.jar -server.port=16000

此时刷新三个注册中心界面，将会看到如下界面

![eureka-registed](/images/spring-cloud/tutorial/eureka-registed.png)

9个服务注册进来了，其中两个消费者服务，分别监听14000和16000端口，两个提供者服务，分别监听9000和15000端口

![monitor-registed](/images/spring-cloud/tutorial/monitor-registed.png)

7个菱形，其中消费者服务(CONSUMER-SERVICE)和提供者服务(PROVIDER-SERVICE)都各有两个实例(instance)

至此，整个项目就运行起来了。

# 结束也是开始

一个相对完整的Spring Cloud项目已经在我们手上诞生了！

然而这并没有结束，仅仅是一个开始。后续还会有非常多的知识在等待着大家，我期待与各位读者共同去探寻Spring Cloud微服务生态的奥秘，增长自己的见闻，完善自身技术。


