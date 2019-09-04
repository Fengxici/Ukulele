---
title: 第六章 服务熔断
date: 2019-08-31 10:04:15
tags: 
- spring cloud
- spring boot
- hystrix
categories:
- spring cloud
- 教程
---

在微服务架构中，每个服务单元都是运行在不同的进程中的，他们之间通过远程调用的方式执行，这样就有可能因为网络原因或是依赖服务自身问题出现调用故障或者延迟。一个服务单元出现故障，很容易因依赖关系而引发故障的蔓延，最终导致整个系统的瘫痪。为了解决这个问题，使得微服务具有更好的稳定性，就产生了断路器等一系列的服务保护机制。

Spring Cloud Hystrix即是实现了断路器、线程隔离等一系列服务保护功能的组件。其目标在于通过控制远程系统、服务和第三方库的节点，从而对延迟和故障提供强大的容错能力。Hystrix具备服务降级、服务熔断、线程和信号格力、请求缓存、请求合并以及服务监控等强大功能。 

-----

接下来我们动手体验。

我们在第四章服务消费者项目的基础上进行升级改造。

## 一 调整下第四章的服务提供者项目
> 1 创建hystrix-demo项目，调整目录结构。

> 2 修改pom依赖。
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>timing.springcloud</groupId>
  <artifactId>hystrix-service</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>hystrix-service</name>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.7.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
      <version>2.1.2.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!-- 增加负载依赖 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
      <version>2.1.2.RELEASE</version>
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

> 3 添加启动类HystrixApplication，与第四章相同，类名有变动。

``` java
@SpringBootApplication
public class HystrixApplication {
    @Bean
    //注册一个具有容错功能的RestTemplate
    @LoadBalanced
    //开启负载均衡客户端
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(HystrixApplication.class, args);
    }
}
```
> 添加controller包并添加HystrixController类，与第四章相同，类名有变动。
``` java
@RestController
public class HystrixController {
    private final RestTemplate restTemplate;
    @Autowired
    public HystrixController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    @GetMapping(value = "ribbon/get")
    public String ribbonGet() {
        return restTemplate.getForEntity("http://PROVIDER-SERVICE/get?para={0}", String.class, "client_ribbon").getBody();
    }
    @PostMapping(value = "ribbon/post")
    public String ribbonPost() {
        HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, Object> parammap = new LinkedMultiValueMap<>();
        parammap.add("para", "ribbon_client");
        HttpEntity<Map> entity = new HttpEntity<>(parammap, headers);
        return restTemplate.postForEntity("http://PROVIDER-SERVICE/post", entity, String.class).getBody();
    }
    @PutMapping(value = "ribbon/put")
    public void ribbonPut() {
        HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, Object> parammap = new LinkedMultiValueMap<>();
        parammap.add("para", "ribbon_client");
        HttpEntity<Map> entity = new HttpEntity<>(parammap, headers);
        restTemplate.put("http://PROVIDER-SERVICE/put", entity);
    }
    @DeleteMapping(value = "ribbon/delete")
    public void ribbonDelete() {
        Map<String, String> params = new HashMap<>();
        params.put("para", "ribbon_client");
        restTemplate.delete("http://PROVIDER-SERVICE/delete/{para}", params);
    }
}
```
> 5 添加配置，与第四章完全相同。

## 二 开始升级改造

> 1 要使用Hystrix，第一步肯定是先引入依赖。
``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency> 
```

> 2 在启动类上添加注解，开启断路器功能
``` java
@EnableCircuitBreaker
public class HystrixApplication {
    ……
}
```
> 3 在HystrixController类中的每个接口方法前添加@HystrixCommand来指定调用失败时的回调方法，改造后的HystrixController如下：
``` java
@RestController
public class HystrixController {
    private final RestTemplate restTemplate;
    @Autowired
    public HystrixController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    @GetMapping(value = "ribbon/get")
    @HystrixCommand(fallbackMethod = "fallback")
    public String ribbonGet() {
        return restTemplate.getForEntity("http://PROVIDER-SERVICE/get?para={0}", String.class, "client_ribbon").getBody();
    }
    @PostMapping(value = "ribbon/post")
    @HystrixCommand(fallbackMethod = "fallback")
    public String ribbonPost() {
        HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, Object> parammap = new LinkedMultiValueMap<>();
        parammap.add("para", "ribbon_client");
        HttpEntity<Map> entity = new HttpEntity<>(parammap, headers);
        return restTemplate.postForEntity("http://PROVIDER-SERVICE/post", entity, String.class).getBody();
    }
    @PutMapping(value = "ribbon/put")
    @HystrixCommand(fallbackMethod = "fallback")
    public void ribbonPut() {
        HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, Object> parammap = new LinkedMultiValueMap<>();
        parammap.add("para", "ribbon_client");
        HttpEntity<Map> entity = new HttpEntity<>(parammap, headers);
        restTemplate.put("http://PROVIDER-SERVICE/put", entity);
    }
    @DeleteMapping(value = "ribbon/delete")
    @HystrixCommand(fallbackMethod = "fallback")
    public void ribbonDelete() {
        Map<String, String> params = new HashMap<>();
        params.put("para", "ribbon_client");
        restTemplate.delete("http://PROVIDER-SERVICE/delete/{para}", params);
    }
    public String fallback() {
        String temp = "testHystrix调用失败！" ;
        System.out.println(temp);
        return temp;
    }
}
```
> 4 观察效果。

启动**注册中心**(第二章，单机版)，**服务提供者**（第三章）。再启动本项目。

访问 http://localhost:9090/ribbon/get 将会得到**client_ribbon_get_service**输出

关闭提供者服务
此时再访问http://localhost:9090/ribbon/get 将会得到
**testHystrix调用失败**的输出

如果没有熔断，情况又是如何呢。猿媛们不妨动手测试一下。

个人能力有限，才疏学浅，如有疏漏，烦请斧正。