---
title: "Spring Filter"
layout: post
author: jsh
tags: Spring
cover: "/assets/cover.jpg"
categories: Spring
---
### Spring Filter

- [Spring Filter](#SpringFilter)
  * [MVC Life Cycle](#MVCLifeCycle)
  * [Filter와 Interceptor 비교](#FilterAndInterceptor)
  * [How to use Filter](#Howto)
  * 사용 가이드
    + [Dependency](#Dependency), [Gradle](#Gradle)
    + [ServletComponentScan](#ServletComponentScan이용)
    + [WebFilter](#WebFilter이용)
    + [Contoller](#TestRestContoller)
    + [Request](#Request)

#### SpringFilter
+ Servlet의 ServletContext 기능으로 사용자에 의해 Servlet이 호출 되기 전/후로 사용자의 요청.응답 헤더 정보 등을 검사 및 설정 할 수 있다.

#### MVCLifeCycle
![Spring MVC](/assets/mvc.png)

#### FilterAndInterceptor
+ Filter는 DispatcherServlet 앞에서 먼저 동작하고, Interceptor는 DispatcherServlet에서 Controllr(Handler) 사이에서 동작한다.
+ Filter
  + Web Application의 Context의 기능
  + Spring 기능을 활용하기에 어려움
  + 일반적으로 인코딩, CORS, XSS, LOG, Certification, Authority 등 을 구현
+ Interceptor
  + Spring의 Spring Context의 기능이며 일종의 Bean
  + Spring Container이기에 다른 Bean을 Injection하여 활용성이 좋음
  + 다른 Bean을 활용 가능하기에 Certification, Authority등을 구현함

#### Howto
+ @ServletComponentScan
  + Application Class에 @ServletComponentScan 선언하고 Filter Configuration Class를 사용하는 방법
+ @WebFilter
  + 각각의 Fliter Class에 @Component, @WebFilter, @Order Annotation을 이용하여 구현하는 방법

#### Dependency

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
#### Gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.projectlombok:lombok:1.18.20'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

#### ServletComponentScan이용

```java
package com.jsh.filter.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@ServletComponentScan
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

```java
package com.jsh.filter.example.config;

import com.jsh.filter.example.filter.FirstSampleFilter;
import com.jsh.filter.example.filter.SecondSampleFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;
import java.util.Arrays;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<Filter> firstSampleFilter(){
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<Filter>(new FirstSampleFilter());
        registrationBean.setUrlPatterns(Arrays.asList("/sample/filter/first"));

        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean<Filter> secondSampleFilter(){
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<Filter>(new SecondSampleFilter());
        registrationBean.setUrlPatterns(Arrays.asList("/sample/filter/second"));

        return registrationBean;
    }
}
```

```java
package com.jsh.filter.example.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
public class FirstSampleFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        logger.info("FirstSampleFilter doFilterInternal() before");
        if (request.getRequestURI().startsWith("/sample/filter/first")) {
            logger.info("uri  startsWith [/sample/filter]");
            response.setStatus(401);
            return;
        }
        filterChain.doFilter(request, response);
        logger.info("FirstSampleFilter doFilterInternal() after");
    }

    @Override
    public void destroy() {
        logger.info("FirstSampleFilter destroy()");
        super.destroy();
    }
}
```

```java
package com.jsh.filter.example.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
public class SecondSampleFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        logger.info("SecondSampleFilter doFilterInternal() before");

        filterChain.doFilter(request, response);

        logger.info("SecondSampleFilter doFilterInternal() after");
    }

    @Override
    public void destroy() {
        logger.info("SecondSampleFilter destroy()");
        super.destroy();
    }
}
```

#### WebFilter이용

```java
package com.jsh.filter.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

```java
package com.jsh.filter.example.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
@WebFilter(urlPatterns = "/sample/filter/first")
@Order(0)
public class FirstSampleFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        logger.info("FirstSampleFilter doFilterInternal() before");
        if (request.getRequestURI().startsWith("/sample/filter/first")) {
            logger.info("uri  startsWith [/sample/filter]");
            response.setStatus(401);
            return;
        }
        filterChain.doFilter(request, response);
        logger.info("FirstSampleFilter doFilterInternal() after");
    }

    @Override
    public void destroy() {
        logger.info("FirstSampleFilter destroy()");
        super.destroy();
    }
}
```

```java
package com.jsh.filter.example.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
@WebFilter(urlPatterns = "/sample/filter/second")
@Order(1)
public class SecondSampleFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        logger.info("SecondSampleFilter doFilterInternal() before");

        filterChain.doFilter(request, response);

        logger.info("SecondSampleFilter doFilterInternal() after");
    }

    @Override
    public void destroy() {
        logger.info("SecondSampleFilter destroy()");
        super.destroy();
    }
}
```

##### TestRestContoller

```java
package com.jsh.filter.example.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FilterController {
    @GetMapping("/sample/filter/first")
    public String first(){
        return "first call";
    }

    @GetMapping("/sample/filter/second")
    public String second(){
        return "sencond call";
    }
}
```

#### Request
+ /sample/filter/first Call

```groovy
2021-04-13 10:58:14.938  INFO 41032 --- [nio-8080-exec-3] c.j.f.example.filter.FirstSampleFilter   : FirstSampleFilter doFilterInternal() before
2021-04-13 10:58:14.938  INFO 41032 --- [nio-8080-exec-3] c.j.f.example.filter.FirstSampleFilter   : uri  startsWith [/sample/filter]
```

```groovy
response Status Code에 401를 반환되게 처리하여 before만 logging 하는 것을 볼 수 있다
```

+ /sample/filter/second Call

```groovy
2021-04-13 11:00:05.545  INFO 41032 --- [nio-8080-exec-6] c.j.f.example.filter.SecondSampleFilter  : SecondSampleFilter doFilterInternal() before
2021-04-13 11:00:05.576  INFO 41032 --- [nio-8080-exec-6] c.j.f.example.filter.SecondSampleFilter  : SecondSampleFilter doFilterInternal() after
```

```groovy
before, after 모두 logging 하는 것을 볼 수 있다
```


github : [https://github.com/suhojang/SpringFilter](https://github.com/suhojang/SpringFilter)
