---
title: "Spring Interceptor"
layout: post
author: jsh
categories: Spring
---

#### Spring Interceptor
+ What is Interceptor?
  + Spring Framework에서 지원하는 기능이며, URI요청, 응답 시점을 가로채서 전/후 처리를 하는 역할을 합니다.
    Interceptor시점에 Spring Context와 Bean에 접근 할 수 있습니다.

+ Interceptor Method
  + PreHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    + Controller에 진입하기 전에 실행 됩니다. 반환 값이 true일 경우 Controller로 진입하고 false일 경우 진입하지 않습니다.
      Object handler는 진입하려는 Controller의 Class 객체가 담겨 있습니다.
  + PostHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
    + Controller 진입 후 View가 Rendering 되기 전에 수행 됩니다.
  + afterComplete(HttpServletRequest request, HttpServletResponse response, Object object, Exception ex)
    + Controller 진입 후 View가 Rendering 된 후에 실행 되는 Method 입니다.
  + afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object h)
    + 비동기 요청 시 PostHandle과 afterComplete가 수행되지 않고, afterConcurrentHandlingStarted가 수행 됩니다.

+ Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
  ```

+ Gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.projectlombok:lombok:1.18.20'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

+ Interceptor 생성

```java
ppackage com.jsh.interceptor.example.interceptor;

import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Slf4j
public class LoggerInterceptor implements HandlerInterceptor {
  private final Logger logger = LoggerFactory.getLogger(LoggerInterceptor.class);

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    logger.info("[" + this.getClass().getName() + "] preHandle");
    return HandlerInterceptor.super.preHandle(request, response, handler);
  }

  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    logger.info("[" + this.getClass().getName() + "] postHandle");
    HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
  }

  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    logger.info("[" + this.getClass().getName() + "] afterCompletion");
    HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
  }
}
```

+ Configuration에 Interceptor 등록

```java
package com.jsh.interceptor.example.config;

import com.jsh.interceptor.example.interceptor.LoggerInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LoggerInterceptor())
            .addPathPatterns("/*")              //include path
            .excludePathPatterns("/example");   //exclude path

    WebMvcConfigurer.super.addInterceptors(registry);
  }
}
```

+ Test를 위한 RestController 생성

```java
package com.jsh.interceptor.example.controller;

import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class InterceptorController {
  private final Logger logger = LoggerFactory.getLogger(InterceptorController.class);

  @GetMapping("/home")
  public String home(){
    logger.info("home call");
    return "home call";
  }

  @GetMapping("/example")
  public String example(){
    logger.info("example call");
    return "example call";
  }
}
```

+ exclude path call

```groovy
2021-04-13 13:40:13.395  INFO 80560 --- [nio-8080-exec-5] c.j.i.e.c.InterceptorController          : example call
```

```groovy
Exclude Path Pattern을 감지하여 Interceptor가 catch 하지 않는다
```

+ include path call

```groovy
2021-04-13 13:41:40.880  INFO 80560 --- [nio-8080-exec-1] c.j.i.e.interceptor.LoggerInterceptor    : [com.jsh.interceptor.example.interceptor.LoggerInterceptor] preHandle
2021-04-13 13:41:40.880  INFO 80560 --- [nio-8080-exec-1] c.j.i.e.c.InterceptorController          : home call
2021-04-13 13:41:40.880  INFO 80560 --- [nio-8080-exec-1] c.j.i.e.interceptor.LoggerInterceptor    : [com.jsh.interceptor.example.interceptor.LoggerInterceptor] postHandle
2021-04-13 13:41:40.881  INFO 80560 --- [nio-8080-exec-1] c.j.i.e.interceptor.LoggerInterceptor    : [com.jsh.interceptor.example.interceptor.LoggerInterceptor] afterCompletion
```

```groovy
Include Path Pattern을 감지하여 Interceptor의 preHandle, postHandle, afterCompletion를 실행하는 것을 볼 수 있다.
```


github : [https://github.com/suhojang/SpringInterceptor](#https://github.com/suhojang/SpringInterceptor)
