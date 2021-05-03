---
title: "SpringBoot Admin"
layout: post
author: jsh
tags: Spring
date: '2021-05-02 10:25:52'
cover: "/assets/cover.jpg"
categories: SpringBoot
---

### Spring Boot Admin

- Spring Boot Admin
  * [개요](#개요)
  * 사용 가이드
    + [Dependency](#Dependency), [Gradle](#Gradle)
    + [EnableAdminServer 설정](#EnableAdminServer)
    + [Security 설정](#Security)
    + [yml 설정](#yml)
    + [Spring Boot Admin Viewer](#SpringBootAdminView)
    + [Spring Boot Admin Client GitHub](https://github.com/suhojang/SpringBootAdminClient)
  * Reference SITE
    + [Spring Boot Admin Reference Guide](https://codecentric.github.io/spring-boot-admin/current/)

#### 개요
+ Spring Boot Admin은 actuator 기반으로 데이터를 분석한다.   
  따라서 Monitor Target은 actuator가 enable 상태여야 하고 적절한 endPoint가 구성 되어 있어야 한다.   
  Spring Boot Admin은 스스로 Target을 탐지하지 않는다.   
  Monitoring Target이 Admin에게 스스로 등록 해야 한다.

#### Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
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

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### Gradle
```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'de.codecentric:spring-boot-admin-starter-server'
implementation 'org.springframework.boot:spring-boot-starter-security'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
testImplementation 'org.springframework.security:spring-security-test'
```

#### EnableAdminServer
````java
package com.jsh.SpringBootAdmin;

import de.codecentric.boot.admin.server.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableAdminServer
public class SpringBootAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootAdminApplication.class, args);
    }

}

````

#### Security
````java
package com.jsh.SpringBootAdmin;

import de.codecentric.boot.admin.server.config.AdminServerProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.web.authentication.SavedRequestAwareAuthenticationSuccessHandler;
import org.springframework.security.web.csrf.CookieCsrfTokenRepository;

@Configuration
public class SecuritySecureConfig extends WebSecurityConfigurerAdapter {
    private final String adminContextPath;

    public SecuritySecureConfig(AdminServerProperties adminContextPath) {
        this.adminContextPath = adminContextPath.getContextPath();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        SavedRequestAwareAuthenticationSuccessHandler successHandler    = new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter("redirectTo");
        successHandler.setDefaultTargetUrl(adminContextPath + "/");

        http.authorizeRequests()
                .antMatchers(adminContextPath + "/assets/**").permitAll()
                .antMatchers(adminContextPath + "/login").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage(adminContextPath + "/login")
                .successHandler(successHandler)
                .and()
                .logout()
                .logoutUrl(adminContextPath + "/logout")
                .and()
                .httpBasic()
                .and()
                .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringAntMatchers(
                        adminContextPath + "/instances"
                        ,adminContextPath + "/actuator/**"
                );
    }
}
````

#### yml
````yml
server:
  port: 8080

spring:
  application:
    name: admin-server
# Spring Boot Admin Login 정보  설정
  security:
    user:
      name: jsh2808
      password: 1234

management:
  endpoints:
    web:
      exposure:
        include: "*"
````

github : [https://github.com/suhojang/SpringBootAdmin](https://github.com/suhojang/SpringBootAdmin)
