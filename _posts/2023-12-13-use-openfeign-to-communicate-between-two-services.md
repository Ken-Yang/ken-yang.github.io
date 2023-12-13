---
layout: post
title: "Use OpenFeign to communicate between two services "
description: ""
category: 
tags: [openfeign]
---
{% include JB/setup %}

{: style="font-size: 16px; line-height: 1.6;"}

這篇是延續前面二篇，把服務連接好以後，如何讓服務之間彼此進行調用？

{: style="font-size: 16px; line-height: 1.6;"}
1. 先[參考上一篇](https://blog.kenyang.net/2023/12/13/springbooteureka-server)，建立1個Eureka Server
2. 先[參考上一篇](https://blog.kenyang.net/2023/12/13/springbooteureka-client)，建立2個Eureka clients，本文範例會創建
   - `調用者` : 1個API service，這API就會對外提供給外部串接，端口設置為8888
   - `被調用者`: 1個Login service，這Login service就是提供給eureka中各個服務進行調用，端口設置為7777

<br />

<!--more-->


{: style="font-size: 16px;line-height: 1.6;"}


---
### 1. 在2個clients專案中創建Interface
---


{: style="font-size: 16px;line-height: 1.6;"}


這個Interface是用來給服務之間進行使用的，<br />

{: style="font-size: 16px;line-height: 1.6;"}
- `調用者` 利用此Interface去調用<br />
- `被調用者` 利用此Interface宣告自己可以被調用的哪些方法，以及必須去 `implements` 此Interface，在裡面實作


{: style="font-size: 16px;line-height: 1.6;"}
因此！在二個eureka client中都需要有此Interface！最好的方式是在一個project中創建interface後，其他需要使用的再去引用他，但這裡為了簡單方便，就粗暴的在二個專案中都放入此interface

{: style="font-size: 16px;line-height: 1.6;"}

Interface如下，其實就是起一個RestFul API地址，其地址是 http://x.x.x.x/servicelogin/login


```java
package net.kenyang;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

/*
 * FeignClient是代表此為一個可以被探索的client
 * value是此服務名稱，其他服務調用需填入此value
 * path就是此服務的api地址
 */
@FeignClient(
        value = "service-login",
        path = "/servicelogin")
public interface ILogin {

    @GetMapping("/login")
    boolean login();
}

```


<br />

---
### 2. 在Login Service專案中實作Interface
---


{: style="font-size: 16px;line-height: 1.6;"}

在 `被調用者` 中去實作上述的Interface

```java
package net.kenyang;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController // 宣告此為一個RestFul API
@RequestMapping(path = "/servicelogin") // path需和Interface中的一致
public class ImplLogin implements ILogin{
    @Override
    public boolean login() {
        // 這裡可以實作實際的業務流程
        return true;
    }
}

```


<br />

---
### 3. 在API Service專案中調用Login service服務
---

#### 3-1. 宣告API Service可以探索及調用其他服務

```java
@SpringBootApplication
@EnableFeignClients(basePackages ={"net.kenyang.interfaces"}) // 宣告可以調用及探索服務，且interface在那個packages底下
public class Main {
...
```

#### 3-2. 創建一個API入口，此API就會調用LoginService

```java
package net.kenyang.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import net.kenyang.interfaces.ILogin;

@RestController
@RequestMapping(path = "/login") // 宣告此API地址的path，也就是 http://x.x.x.x/login
public class Login {

    @Autowired // 會自動注入，讓spring完成bean自動封裝
    ILogin login;

    @GetMapping("/")
    public boolean index() {
        return login.login();
    }
}

```

<br />

---
### 4. 加入OpenFeign的dependcy
---

{: style="font-size: 16px;line-height: 1.6;"}

在二個專案中都要引入

```

        <!-- 使用openfeign做服務探索 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>4.1.0</version>
        </dependency>
```


<br />

---
### 5. Run
---

{: style="font-size: 16px;line-height: 1.6;"}

將上述二個服務都註冊上Eureka Server後， <br />
訪問API的地址 http://127.0.0.1:8888/login <br />
就會顯示service吐回來的結果











