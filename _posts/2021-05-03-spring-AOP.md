---
title: "Spring AOP"
layout: post
author: jsh
tags: Spring
cover: "/assets/cover.jpg"
categories: Spring
---

#### Spring AOP
+ 언제 사용되는가?
  + 성능 검사
  + 트랜잭션 처리
  + 로깅
  + 예외 반환
  + 검증


```
실 예로, @Transactional, @Cache같은 애노테이션들은 AOP를 활용하여 동작하게 된다.
```

+ 구성요소
  + JoinPoint: 모듈의 기능이 삽입되어 동작할 수 있는 실행 가능한 특정 위치
  + PointCut: 어떤 클래스의 어느 JoinPoint를 사용할 것인지를 결정
  + Advice: 각 JoinPoint에 삽입되어져 동작할 수 있는 코드
  + Interceptor: InterceptorChain 방식의 AOP 툴에서 사용하는 용어로 주로 한개의 호출 메소드를 가지는 Advice
  + Weaving: PointCut에 의해서 결정된 JoinPoint에 지정된 Advice를 삽입하는 과정(CrossCutting)
  + Introduction: 정적인 방식의 AOP 기술
  + Aspect: PointCut + Advice + (Introduction)

+ Advice 종류
  + Before : 타겟의 메서드가 실행되기 이전(before) 시점에 처리해야 할 필요가 있는 부가기능을 정의한다. Joinpoint 앞에서 실행되는 Advice
  + After returning : 타겟의 메서드가 정상적으로 실행된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의한다. Jointpoint 메서드 호출이 정상적으로 종료된 뒤에 실행되는 Advice
  + After throwing : 타겟의 메서드가 예외를 발생된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의한다. 예외가 던져질 때 실행되는 Advice
  + After : 메서드가 정상적으로 실행되는지 또는 예외를 발생시키는지 여부에 상관없이 어드바이스를 정의한다. Joinpoint 가 정상적으로 종료된 여부에 관계 없이 항상 실행되는 Advice
  + Around : 타겟의 메서드가 호출되기 이전(before) 시점과 이후 (after) 시점에 모두 처리해야 할 필요가 있는
    부가기능을 정의한다. Joinpoint 앞과 뒤에서 실행되는 Adcvice
  + Introdution : 클래스에 인터페이스와 구현을 추가하는 특수한 Advice

+ Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
  ```

+ Gradle

```groovy
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.springframework.boot:spring-boot-starter-test')
compile('org.springframework.boot:spring-boot-starter-aop')

```

+ PointCut && JoinPoints
  | Pointcut | JoinPoints |
  | ----- | ----- |
  | execution(public * *(..)) | public 메소드 실행 |
  | execution(* set*(..)) | 이름이 set으로 시작하는 모든 메소드명 실행 |
  | execution(* get*(..)) | 이름이 get으로 시작하는 모든 메소드명 실행 |
  | execution(* com.jsh.service.TestAopService.*(..)) | TestAopService 인터페이스의 모든 메소드 실행 |
  | execution(* com.jsh.service.*.*(..)) | service 패키지의 모든 메소드 실행 |
  | execution(* com.jsh.service..*.*(..)) | service 패키지와 하위 패키지의 모든 메소드 실행 |
  | within(com.jsh.service.*) | service 패키지 내의 모든 결합점 (클래스 포함) |
  | within(com.jsh.service..*) | service 패키지 및 하위 패키지의 모든 결합점 (클래스 포함) |
  | bean(*JSHRepository) | 이름이 “JSHRepository”로 끝나는 모든 빈 |
  | bean(*) | 모든 빈 |
  | bean(jsh*) | 이름이 'jsh'로 시작되는 모든 빈 |

+ Config 파일을 생성한 후 @Component로 선언하고 @EnableAspectJAutoProxy를 적용하여 AOP를 찾을 수 있게 처리

```java
package com.jsh.aop.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
public class AspectJConfig {
}
```

+ 공통기능을 정의하고 공통 기능이 사용 될 시점을 정의

```java
package com.jsh.aop.component;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TestAspectJ {
    private static final Logger logger   = LoggerFactory.getLogger(TestAspectJ.class);

    @Before("execution(* com.jsh.aop.service.*.*Aop(..))")
    public void onBeforeHandler(JoinPoint joinpoint){
        logger.info("=============== onBeforeThing");
    }
    @After("execution(* com.jsh.aop.service.*.*Aop(..))")
    public void onAfterHandler(JoinPoint joinpoint){
        logger.info("=============== onAfterHandler");
    }

    @AfterReturning(pointcut = "execution(* com.jsh.aop.service.*.*Aop(..))", returning = "str")
    public void onAfterReturningHandler(JoinPoint joinpoint, Object str){
        logger.info("@AfterReturning : " + str);
        logger.info("=============== onAfterReturningHandler");
    }

    @Around("execution(* com.jsh.aop.service.*.*Aop(..))")
    public Object onAround(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("=============== onAroundHandler before");

        Object result   = joinPoint.proceed();

        logger.info("=============== onAroundHandler after");

        return result;
    }
}
```

+ Controller에 URI를 정의하고 AOP가 작동 할 수 있게 Service 생성

```java
package com.jsh.aop.controller;

import com.jsh.aop.service.TestAopService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestAopController {
    @Autowired
    private TestAopService service;

    @GetMapping(value="/noAop")
    public String noAop(){
        return service.test();
    }

    @GetMapping(value="/aop")
    public String aop(){
        return service.testAop();
    }
}
```

```java
package com.jsh.aop.service.impl;

import com.jsh.aop.service.TestAopService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class TestAopServiceImpl implements TestAopService {
    private static final Logger logger  = LoggerFactory.getLogger(TestAopServiceImpl.class);

    @Override
    public String test() {
        String msg  = "Hello, Spring Boot No AOP";
        logger.info(msg);
        return msg;
    }

    @Override
    public String testAop() {
        String msg  = "Hello, Spring Boot AOP";
        logger.info(msg);
        return msg;
    }
}
```

+ AOP URI Call - http://localhost:8080/aop

```
2021-04-12 12:37:19.745  INFO 24164 --- [nio-8080-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-04-12 12:37:19.745  INFO 24164 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2021-04-12 12:37:19.746  INFO 24164 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
2021-04-12 12:37:19.765  INFO 24164 --- [nio-8080-exec-2] com.jsh.aop.component.TestAspectJ        : =============== onAroundHandler before
2021-04-12 12:37:19.765  INFO 24164 --- [nio-8080-exec-2] com.jsh.aop.component.TestAspectJ        : =============== onBeforeThing
2021-04-12 12:37:19.774  INFO 24164 --- [nio-8080-exec-2] c.j.aop.service.impl.TestAopServiceImpl  : Hello, Spring Boot AOP
2021-04-12 12:37:19.774  INFO 24164 --- [nio-8080-exec-2] com.jsh.aop.component.TestAspectJ        : @AfterReturning : Hello, Spring Boot AOP
2021-04-12 12:37:19.774  INFO 24164 --- [nio-8080-exec-2] com.jsh.aop.component.TestAspectJ        : =============== onAfterReturningHandler
2021-04-12 12:37:19.774  INFO 24164 --- [nio-8080-exec-2] com.jsh.aop.component.TestAspectJ        : =============== onAfterHandler
2021-04-12 12:37:19.774  INFO 24164 --- [nio-8080-exec-2] com.jsh.aop.component.TestAspectJ        : =============== onAroundHandler after
```

+ NO AOP URI Call - http://localhost:8080/noAop

```
2021-04-12 12:38:17.990  INFO 24164 --- [nio-8080-exec-4] c.j.aop.service.impl.TestAopServiceImpl  : Hello, Spring Boot No AOP
```

github : [https://github.com/suhojang/SpringSecurity](https://github.com/suhojang/SpringSecurity)
