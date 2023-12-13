---
layout: post
title: "使用SpringBoot創建Eureka Server"
description: ""
category: 
tags: [eureka, springboot]
---
{% include JB/setup %}

{: style="font-size: 16px;line-height: 2;"}

這篇是說明如何使用SpringBoot 3.2.0和Eureka 4.1.0創建一個Eureka Server

---
### 1. 打開Intelli J IDEA
---

{: style="font-size: 16px;line-height: 1.6;"}

- 點擊 `File` 
- 點擊 `New`
- 點擊 `Project`

<br />

<!--more-->

---
### 2. 編輯pom.xml，此步驟只是把spring相關的dependecy抓下來
---


{: style="font-size: 16px;line-height: 1.6;"}

- 去網站 https://start.spring.io/ 生成springboot的xml文件
- `注意！我用的是spring 3.2.0，所以要求Java至少要17以上，所以記得選對JAVA SDK` 
   -  下載對應SDK，去 `Project Setting -> Project` 
      -  選擇`SDK` 這裡的下拉選單中選 `Download JDK` ，選擇 `Oracje JDK 21`
      -  點選`Language Level`，選擇 `21`
   - 設定 Java compiler，在 MAC中按 `cmd+,` 
      -  點選`Build, Execution, Deployment` -> `Compiler` -> `Java Compiler` 
      -  在對應module中更改 `Target bytecode version`


{: style="font-size: 16px;line-height: 1.6;"}

pom.xml範例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>net.kenyang.server</groupId>
    <artifactId>employee-lottery-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>employee-lottery-api</name>
    <description>員工抽獎API</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
<!--            <artifactId>spring-boot-starter-web</artifactId> --> <!-- 這個是單純給restful，但因為會有eureka，所以不用 -->
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
## 3. 編輯pom.xml，此步驟把Eureka相關的dependecy補上
---


{: style="font-size: 16px;line-height: 1.6;"}

- 切記！版本很重要！會有相依性！

```xml
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>4.1.0</version>
        </dependency>
```


<br />

---
## 4. (optional) 編輯pom.xml，此步驟把登入模組的dependcy補上
---


{: style="font-size: 16px;line-height: 1.6;"}

為什麼optional呢？因為我發現eureka `client` 在某個版本後，沒有去實作 authorization header 當連結到server時，因此如果設置了這段，在client那邊必須要去改寫AbstractJerseyEurekaHttpClient。

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

<br />

---
# 5. 修改代碼
---

```java
package net.kenyang.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer // 宣告為一個EurekaServer
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }


}
```

<br />

---
# 6. 配置檔
---


{: style="font-size: 16px;line-height: 1.6;"}

在resources資料夾下創建 `application.yml`

```
server:
  port: 8886
spring:
  security: # 選填，如果有spring-boot-starter-security才需要
    user:
      name: admin
      password: admin
eureka:
  instance:
    hostname: ${spring.cloud.client.ip-address}
    preferIpAddress: true  # 使用IP rather than HOst
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

<br />

---
# 7. Run
---


{: style="font-size: 16px;line-height: 1.6;"}

點選compile後，再點選run，
去瀏覽器看 127.0.0.1:8886，就會看到管理介面


<br />

<br />

<br />