---
layout: post
title: "使用SpringBoot創建Eureka Client"
description: ""
category: 
tags: [spring, eureka]
---
{% include JB/setup %}


{: style="font-size: 16px;line-height: 1.6;"}

這篇會教學如何創建一個Eureka Client，並且連到先前創的Eureka Server上

---
### 1. 打開Intelli J IDEA
---

- 點擊 `File` 
- 點擊 `New`
- 點擊 `Project`


<br />

<!--more-->

---
### 2. 編輯pom.xml，此步驟只是把spring相關的dependecy抓下來
---

- 去網站 https://start.spring.io/ 生成springboot的xml文件
- `注意！我用的是spring 3.2.0，所以要求Java至少要17以上，所以記得選對JAVA SDK` 
   -  下載對應SDK，去 `Project Setting -> Project` 
      -  選擇`SDK` 這裡的下拉選單中選 `Download JDK` ，選擇 `Oracje JDK 21`
      -  點選`Language Level`，選擇 `21`
   - 設定 Java compiler，在 MAC中按 `cmd+,` 
      -  點選`Build, Execution, Deployment` -> `Compiler` -> `Java Compiler` 
      -  在對應module中更改 `Target bytecode version`

      


pom.xml範例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>tech.starlink</groupId>
    <artifactId>employee-lotter-service-login</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>employee-lottery-service-login</name>
    <description>員工抽獎登入服務</description>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
<!--            <artifactId>spring-boot-starter</artifactId>-->
            <!-- eureka client一定要使用 starter-web，不能單純使用starter -->
            <artifactId>spring-boot-starter-web</artifactId> 
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>4.1.0</version>
        </dependency>

        <!-- 以下二個是health check需使用的套件 /actuator/info, /actuator/health -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-actuator-autoconfigure</artifactId>
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

<br />


---
### 3. 宣告成為一個Eureka Client
---


```java
package tech.starlink;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// 在eureka 4.1.0，就不用特別宣告他是一個eureka client
@SpringBootApplication
public class Main {

    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }
}
```

<br />

---
### 4. 配置application.yml
---


```xml
server:
  port: 8887
spring:
  application:
    name: service-login
eureka:
  instance:
    hostname: ${spring.cloud.client.ip-address}
    preferIpAddress: true # 使用IP rather than HOst
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://127.0.0.1:8888/eureka/
    healthcheck: true
    lease:
      duration: 5
management:
  endpoints:
    web:
      exposure: # 在/actuator路徑下旨暴露info和health
        include: "info, health"
```


<br />

---
## 5.  Run
---

- 記得先clean 和compile
- 啟動後去server UI上就會看到此服務註冊上去

