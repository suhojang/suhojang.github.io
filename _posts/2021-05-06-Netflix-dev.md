---
title: "Spring Cloud Netfilx를 이용한 MSA 구축"
layout: post
author: jsh
cover: "/assets/cover.jpg"
categories: Spring Cloud
---

Spring Cloud Netfilx를 이용한 MSA 구축
---

## 1. Eureka(Service Discovery) 서버 구축

### 1) gradle 추가

```groovy
plugins {
  id 'org.springframework.boot' version '2.3.10.RELEASE'
  id 'io.spring.dependency-management' version '1.0.11.RELEASE'
  id 'java'
}

group = 'com.jsh'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
  mavenCentral()
}

ext {
  set('springCloudVersion', "Hoxton.SR10")
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}

test {
  useJUnitPlatform()
} 
```

### 2) application.yml 설정

```yaml
server:
  port: 8761  # server의 port 지정

spring:
  application:
    name: ecureka-server # application name 설정

eureka:
  instance:
    hostname: localhost
    prefer-ip-address: true
  client:
    register-with-eureka: true # false 자기 자신을 서비스에 등록하지 않는다. true 자기 자신에게 계속 Health check를 하게 됨
    fetch-registry: true # 로컬 캐싱여부
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```


### 3) Eureka 서버 지정
> Application java 파일에 @EnableEurekaServer 지정하면 끝.

```java
package com.jsh.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}
```

## 2. Eureka Client 구축하기

> 여기서는 display, product 2개의 프로젝트를 클라이언트 지정 하였습니다.   
> 해당 프로젝트는 hystrix 와 open feign 을 이용한 프로젝트로 생성 되었습니다.

### 2.1 Display Project 생성

### 1) gradle 추가(display)

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-ribbon'
	implementation 'org.springframework.retry:spring-retry:1.2.2.RELEASE'
	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
	implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
}
```

### 2) application.yml 설정

```yaml
server:
  port: 8081

spring:
  application:
    name: display

# feign 별 hystrix 설정
hystrix:
  command:
    productInfo: # commnad Key. use 'default' for global setting.
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000 # default 1000ms
      circuitBreaker:
        requestVolumeThreshold: 1 # default 20
        errorThresholdPercentage: 50 # default 50
    FeignProductRemoteService#getProductInfo(String):
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000 # default 1000ms
      circuitBreaker:
        requestVolumeThreshold: 1 # default 20
        errorThresholdPercentage: 50 # default 50

# hystrix 설정
# hystrix:
#   command:
#     default:
#       execution:
#         isolation:
#           thread:
#             timeoutInMilliseconds: 3000 # default 1000ms
#       circuitBreaker:
#         requestVolumeThreshold: 10 # default 20
#         errorThresholdPercentage: 50 # default 50

# ribbon설정(default round robin방식)
product:
  ribbon:
# listOfServers가 없는 경우 자동으로 Eureka Server에서 Server List 가져옴
#    listOfServers: localhost:8082, localhost:7777
    MaxAutoRetries: 0
    MaxAutoRetriesNextServer: 1

eureka:
  instance:
    prefer-ip-address: true # ip address base 등록
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka, http://localhost:8762/eureka  # default address

feign:
  hystrix:
    enabled: true
```

### 3) Eureka 클라이언트 지정
> Application java 파일에 @EnableEurekaClient 지정하면 끝.    
> 해당 프로젝트는 CircuitBreaker와 FeignClient를 구현하기 위해서 추가 Annotation 을 지정 하였습니다.

```java
package com.jsh.hystrix;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableCircuitBreaker
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class DisplayApplication {
	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(DisplayApplication.class, args);
	}
}
```


### 4) 테스트를 위한 파일 생성

#### 4.1) RestConroller 생성

```java
package com.jsh.hystrix.api;

import com.jsh.hystrix.service.FeignProductRemoteService;
import com.jsh.hystrix.service.ProductRemoteService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(path = "/displays")
public class DisplayController {
    @Autowired
    private ProductRemoteService productRemoteService;

    @Autowired
    private FeignProductRemoteService feignProductRemoteService;


    @GetMapping(path = "/{displayId}")
    public String getDisplayDetail(@PathVariable String displayId){
        String productInfo  = getProductInfo();
        return String.format("[display id = %s at %s %s]", displayId, System.currentTimeMillis(), productInfo);

    }

    private String getProductInfo(){
//        return productRemoteService.getProductInfo("12345");
        return feignProductRemoteService.getProductInfo("12345");
    }
}
```

#### 4.2) Feign 처리를 위한 Feign Client 생성

```java
package com.jsh.hystrix.service;

import com.jsh.hystrix.factory.FeignProductRemoteServiceFallbackFactory;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * FeignClient설정 시 application명으로 접근이 가능하고 FallbackFactory설정으로 circuitBreaker 기능을 사용할 수 있다.
 * application config 파일에 feign.hystrix.enabled=true 설정 필요
 *
 * @FeignClient(name = "[spring.application.name]으로 등록 된 Application명 지정")
 * default 로드밸런싱(Eureka의 Ribbon)
 * 직접 url지정을 원할 경우는 @FeignClient(name = "product", url= "지정 url")형태로 사용
 * 다른 Application과 통신을 할 때 유용
 *
 * Zuul Api Gateway Server 송신하는 방식
 * @FeignClient(name = "[zuul server의 application name]", fallbackFactory = FeignProductRemoteServiceFallbackFactory.class)
 *
 * eureka server를 통해 송신하는 방식
 * @FeignClient(name = "[eureka server에 등록 된 application name]", fallbackFactory = FeignProductRemoteServiceFallbackFactory.class)
 */
//@FeignClient(name = "product", fallbackFactory = FeignProductRemoteServiceFallbackFactory.class)
@FeignClient(name = "zuul", fallbackFactory = FeignProductRemoteServiceFallbackFactory.class)
public interface FeignProductRemoteService {
    @RequestMapping("/products/{productId}")
    String getProductInfo(@PathVariable("productId") String productId);
}
```

#### 4.3) Feign Fallback 처리를 위한 Fallback Factory 생성

```java
package com.jsh.hystrix.factory;

import com.jsh.hystrix.service.FeignProductRemoteService;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;

@Component
public class FeignProductRemoteServiceFallbackFactory implements FallbackFactory<FeignProductRemoteService> {
    @Override
    public FeignProductRemoteService create(Throwable cause) {
        System.out.println("t = " + cause);
        return productId -> "[This Product is sold out]";
    }
}
```

<br>

### 2.2 Product Project 생성

### 1) gradle 추가(product)

```groovy
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}
```

### 2) application.yml 설정

```yaml
server:
  port: 8082

spring:
  application:
    name: product

eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka, http://localhost:8762/eureka  # default address
```

### 3) Eureka 클라이언트 지정
> Application java 파일에 @EnableEurekaClient 지정하면 끝.    

```java
package com.jsh.hystrix;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class ProductApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProductApplication.class, args);
	}

}
```

### 4) 테스트를 위한 파일 생성

```java
package com.jsh.hystrix.api;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
@RequestMapping(path = "/products")
public class ProductController {
  @GetMapping(path = "{productId}")
  public String getProductInfo(@PathVariable String productId){
//        throw new RuntimeException("I/O Exception");
    log.info("productId = " + productId);
    return "[product id = " + productId + " at " + System.currentTimeMillis() + "]";
  }
}
```


## 3. API Gateway 서버(Zuul) 구축

### 1) gradle 추가

```groovy
dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
  implementation 'org.springframework.retry:spring-retry:1.2.2.RELEASE'
  implementation 'org.springframework.boot:spring-boot-actuator'
}
```

### 2) application.yml 설정

```yaml
spring:
  application:
    name: zuul
#  profiles:
#    active: default         # 서비스가 실행할 기본 프로파일
#    cloud:
#      config:
#        uri: http://localhost:8889  # Config Server 위치
# cloud config git주소에 [서비스 ID]폴더 생성 후  profile생성

server:
  port: 8765

management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    shutdown:
      enabled: true

zuul:
# ignored-services: 'product'   # 자동 경로 매핑 무시, 쉼표로 한 번에 여러 서비스 제외 가능
  ignored-services: '*'   # 유레카 기반 모든 경로 제외
# prefix: /api            # 정의한 모든 서비스에 /api 접두어
  routes:
    product:
      path: /products/**
      serviceId: product
      stripPrefix: false
    display:
      path: /displays/**
      serviceId: display
      stripPrefix: false
  ribbon-isolation-strategy: thread
  thread-pool:
    use-separate-thread-pools: true
    thread-pool-key-prefix: zuul-

eureka:
  instance:
    non-secure-port: ${server.port}
    prefer-ip-address: true
  client:
    register-with-eureka: true # Eureka Server에 서비스 등록
    fetch-registry: true # 레지스트리 정보를 로컬에 캐싱
    service-url:
      defaultZone: http://localhost:8761/eureka, http://localhost:8762/eureka

# apllication 별 hystrix설정
hystrix:
  command:
# default hystrix 설정
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000
# application 별 hytrix 설정
    product:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 12000 # hystrix timeout 12sec로 설정 (default 1sec, ribbon 의 timeout 보다 커야 기대하는 대로 동작함)
# application 별 threadpool 설정
# 위에 정의 한 thread-pool-key-prefix 명 뒤에 zuul routes명이 붙는다.
  threadpool:
    zuul-product:
      coreSize: 1
      maximumSize: 2
      allowMaximumSizeToDivergeFromCoreSize: true
    zuul-display:
      coreSize: 1
      maximumSize: 2
      allowMaximumSizeToDivergeFromCoreSize: true

# application 별 ribbon timeout 설정
product:
  ribbon:
    MaxAutoRetriesNextServer: 1
    ReadTimeout: 3000
    ConnectTimeout: 3000
    MaxTotalConnections: 30
    MaxConnectionsPerHost: 10

display:
  ribbon:
    MaxAutoRetriesNextServer: 1
    ReadTimeout: 3000
    ConnectTimeout: 3000
    MaxTotalConnections: 30
    MaxConnectionsPerHost: 10
```

### 3) Zuul Proxy 지정

> @EnableZuulProxy 지정하면 끝.   
> @EnableDiscoveryClient 를 지정하여 Eureka 서버에 등록

```java
package com.jsh.zuul;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class ZuulApplication {

	public static void main(String[] args) {
		SpringApplication.run(ZuulApplication.class, args);
	}

}
```

## 4. 결과 확인

> 테스트는 Eureka 서버 2대(port: 8761, 8762), Zuul 서버 1대(port: 8765), Product 서버 2대(port: 8082, 8083), Display 서버 1대(port: 8081)로 테스트 진행 하였습니다.

### 4.1 Eureka 서버 확인

+ Eureka 서버1

![/assets/eureka-1.png](/assets/eureka-1.png)

+ Eureka 서버2

![/assets/eureka-2.png](/assets/eureka-2.png)


위 그림을 보면 Eureka 서버1(8761)에 Eureka Client들이 붙은 걸 확인 할 수 있다.   
Eureka 서버2(8762)는 Eureka Client들이 붙지 않은 걸 볼 수 있는데 Eureka 서버1이 장애(Down) 시 아래 Log처럼 
Eureka 서버2(8762)로 retry하는 걸 볼 수 있다.

```groovy
2021-05-06 17:07:31.586  INFO 24932 --- [trap-executor-0] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration
2021-05-06 17:08:08.671  INFO 24932 --- [freshExecutor-0] c.n.d.s.t.d.RedirectingEurekaHttpClient  : Request execution error. endpoint=DefaultEndpoint{ serviceUrl='http://localhost:8762/eureka/} exception=java.net.ConnectException: Connection refused: connect stacktrace=com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused: connect
  at com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpClient4Handler.java:187)
  at com.sun.jersey.api.client.filter.GZIPContentEncodingFilter.handle(GZIPContentEncodingFilter.java:123)
  at com.netflix.discovery.EurekaIdentityHeaderFilter.handle(EurekaIdentityHeaderFilter.java:27)
  at com.sun.jersey.api.client.Client.handle(Client.java:652)
  at com.sun.jersey.api.client.WebResource.handle(WebResource.java:682)
  at com.sun.jersey.api.client.WebResource.access$200(WebResource.java:74)
  at com.sun.jersey.api.client.WebResource$Builder.get(WebResource.java:509)
  at com.netflix.discovery.shared.transport.jersey.AbstractJerseyEurekaHttpClient.getApplicationsInternal(AbstractJerseyEurekaHttpClient.java:196)
  at com.netflix.discovery.shared.transport.jersey.AbstractJerseyEurekaHttpClient.getDelta(AbstractJerseyEurekaHttpClient.java:172)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$7.execute(EurekaHttpClientDecorator.java:152)
  at com.netflix.discovery.shared.transport.decorator.MetricsCollectingEurekaHttpClient.execute(MetricsCollectingEurekaHttpClient.java:73)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.getDelta(EurekaHttpClientDecorator.java:149)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$7.execute(EurekaHttpClientDecorator.java:152)
  at com.netflix.discovery.shared.transport.decorator.RedirectingEurekaHttpClient.execute(RedirectingEurekaHttpClient.java:91)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.getDelta(EurekaHttpClientDecorator.java:149)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$7.execute(EurekaHttpClientDecorator.java:152)
  at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:120)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.getDelta(EurekaHttpClientDecorator.java:149)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$7.execute(EurekaHttpClientDecorator.java:152)
  at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77)
  at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.getDelta(EurekaHttpClientDecorator.java:149)
  at com.netflix.discovery.DiscoveryClient.getAndUpdateDelta(DiscoveryClient.java:1135)
  at com.netflix.discovery.DiscoveryClient.fetchRegistry(DiscoveryClient.java:1016)
  at com.netflix.discovery.DiscoveryClient.refreshRegistry(DiscoveryClient.java:1531)
  at com.netflix.discovery.DiscoveryClient$CacheRefreshThread.run(DiscoveryClient.java:1498)
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
  at java.util.concurrent.FutureTask.run(FutureTask.java:266)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
  Caused by: java.net.ConnectException: Connection refused: connect
  at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
  at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
  at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
  at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
  at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
  at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
  at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
  at java.net.Socket.connect(Socket.java:589)
  at org.apache.http.conn.scheme.PlainSocketFactory.connectSocket(PlainSocketFactory.java:121)
  at org.apache.http.impl.conn.DefaultClientConnectionOperator.openConnection(DefaultClientConnectionOperator.java:180)
  at org.apache.http.impl.conn.AbstractPoolEntry.open(AbstractPoolEntry.java:144)
  at org.apache.http.impl.conn.AbstractPooledConnAdapter.open(AbstractPooledConnAdapter.java:134)
  at org.apache.http.impl.client.DefaultRequestDirector.tryConnect(DefaultRequestDirector.java:605)
  at org.apache.http.impl.client.DefaultRequestDirector.execute(DefaultRequestDirector.java:440)
  at org.apache.http.impl.client.AbstractHttpClient.doExecute(AbstractHttpClient.java:835)
  at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:118)
  at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56)
  at com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpClient4Handler.java:173)
  ... 29 more

2021-05-06 17:08:08.671  WARN 24932 --- [freshExecutor-0] c.n.d.s.t.d.RetryableEurekaHttpClient    : Request execution failed with message: java.net.ConnectException: Connection refused: connect
2021-05-06 17:08:08.690  INFO 24932 --- [freshExecutor-0] c.n.d.s.t.d.RetryableEurekaHttpClient    : Request execution succeeded on retry #1
```


+ Eureka 서버1을 임의로 Down 후 Eureka 서버2의 모습

![/assets/eureka-3.png](/assets/eureka-3.png)


+ API Gateway(Zuul)를 이용한 Rest 통신 테스트(postman 이용)

![/assets/apigateway-test01.png](/assets/apigateway-test01.png)

위 그림과 같이 정상적으로 데이터를 응답 받는 것을 확인 할 수 있다.


## 결론

몇 가지 간단한 어노테이션을 사용하여 어플리케이션 내부의 공통 패턴을 신속하게 사용하고 설정할 수 있습니다.   
그리고 battle-tested를 거친 Netflix component를 통해 대규모 분산 시스템을 구축할 수 있습니다.

   
