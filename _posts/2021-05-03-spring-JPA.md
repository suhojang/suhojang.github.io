---
title: "Spring JPA"
layout: post
author: jsh
tags: Spring
date: '2021-05-02 10:25:52'
cover: "/assets/cover.jpg"
categories: Spring
---

### Spring JPA

- Spring JPA
  * [ORM](#ORM)
  * [JPA](#JPA)
    + [장점](#장점)
    + [단점](#단점)
  * [Hibernate](#Hibernate)
  * 사용 가이드
    + [Dependency](#Dependency), [Gradle](#Gradle)
    + [Entity](#Entity생성)
    + [Repository](#Repository생성)
    + [Service](#Service생성)
    + [RestController](#RestController생성)
    + [GET](#GET), [POST](#POST), [PUT](#PUT), [DELETE](#DELETE)
  * Reference SITE
    + [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
    + [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)
    + [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
    + [Spring Data JPA Document](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#preface)

#### ORM
+ Object-Relational Mapping(객체와 관계형 데이터베이스 Mapping, 객체와 DB의 Table이 Mapping을 이루는 것)
+ 객체가 Table이 되도록 Mapping 시켜주는 Framework 이다.
+ 프로그램의 복잡도를 줄이고 JAVA Object와 Query를 분리할 수 있으며 Transaction 처리나기타 Database 관련 작업들을 좀 더 편리하게 처리 할 수 있는 방법
+ SQL Query가 아닌 직관적인 Code(Method)로서 Data를 조작 할 수 있다.
  + before : SELECT * FROM USER;
  > 위 Query를 ORM을 이용하면 다음과 같다.
  + after : user.findAll()

#### JPA
+ Java Persistence API(JAVA ORM 기술에 대한 API 표준 명세)
+ 한마디로 ORM을 사용하기 위한 Interface를 모아둔 것이라고 볼 수 있다.
+ Java Application에서 관계형 DataBase를 사용하는 방식을 정의한 Interface이다.
+ ORM에 대한 Java API 규격이며 Hibernate, OpenJPA등이 JPA를 구현 한 구현체이다.
+ Hibernate 이외에도 EclipseLink, DataNucleus, OpenJPA, TopLink 등이 있다.
> 결국 Interface이기 때문에 JPA를 사용하기 위해서는 JPA를 구현한 Hibernate, EclipseLink, DataNucleus, OpenJPA, TopLink같은 ORM Framework를 사용해야 한다.

#### Hibernate
+ JPA를 사용하기 위한 ORM Framework 중 하나.
+ Hibernate는 JPA명세의 구현체이다.
  > javax.persistence.EntityManager와 같은 JPA의 Interface를 직접 구현한 library 이다.

#### 장점
+ 생산성이 뛰어나고 유지보수가 용이하다.
  + 객체 지향적인 코드로 인해 직관적이고 비즈니스 로직에 좀 더 집중할 수 있게 도와준다.
  + 객체 지향적으로 데이터를 관리할 수 있기 때문에 전체 프로그램 구조를 일관되게 유지 할 수 있다.
  + SQL을 직접적으로 작성하지 않고 객체를 사용하여 동작하기 때문에 유지보수 비용이 적다.
  + DB 컬럼이 추가 될 때마다 Table 수정이나 SQL 수정하는 과정이 줄어들고 값을 할당하거나 변수 선언등의 부수적인 코드 또한 급격히 줄어든다.
  + 각각의 객체에 대한 코드를 별도로 작성하여 코드의 가독성이 증가한다.

+ DBMS에 대한 종속성이 줄어든다.
  + DBMS가 변경된다 하더라도 Source, Query, 구현 방법, 자료형 Type등을 변경 할 필요가 없다.
  + 개발자는 Object에만 집중하면 되고 DBMS를 교체하는 작업에도 비교적 적은 리스크와 시간이 소요 된다.

#### 단점
+ JPA의 장점을 살려 잘 사용하려면 학습 비용이 높은 편이다.
+ 복잡한 Query를 사용해야 할 때에 불리하다.
  + 업무 비즈니스가 매우 복잡한 경우 JPA로 처리하기 어렵고 통계처리와 같은 복잡한 쿼리 자체를 ORM으로 표현하는데 한계가 있다.
  + 결국 기존 DataBase 중심으로 되어 있는 환경에서는 JPA를 사용하기도 어렵고 힘을 발휘하기 어렵다.
  + 잘못 사용 할 경우 실제 SQL문을 직접 작성하는 것보다는 성능이 비교적 떨어 질 수 있다.
  + 대용량 데이터 기반의 환경에서도 튜닝이 어려워 상대적으로 기존 방식보다 성능이 떨어질 수 있다.

#### Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### Gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-web'
compileOnly 'org.projectlombok:lombok'
runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
annotationProcessor 'org.projectlombok:lombok'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

#### Entity생성

````java
package com.jsh.jpaExample.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import javax.persistence.*;
import java.time.LocalDateTime;

@NoArgsConstructor
@Setter
@Getter
@Table(name = "book")
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String category;

    @Column(nullable = false)
    private long sellCount;

    @CreationTimestamp
    @Column(nullable = false, length = 20, updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    @Column(length = 20)
    private LocalDateTime updatedAt;

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", category='" + category + '\'' +
                ", sellCount=" + sellCount +
                ", createdAt=" + createdAt +
                ", updatedAt=" + updatedAt +
                '}';
    }
}
````

#### Repository생성

````java
package com.jsh.jpaExample.repository;

import com.jsh.jpaExample.entity.Book;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BookRepository extends CrudRepository<Book, Long> {
    public Book findBookByName(String name);
    public Book findBookByCategory(String category);
}
````

#### Service생성

````java
package com.jsh.jpaExample.service;

import com.jsh.jpaExample.entity.Book;

import java.util.List;

public interface BookService {
    public Book findBookByName(String name);
    public Book findBookByCategory(String category);
    public Book save(Book book);
    public List<Book> findBookAll();
    public void delete(Long id);
}
````

````java
package com.jsh.jpaExample.service;

import com.jsh.jpaExample.entity.Book;
import com.jsh.jpaExample.repository.BookRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service("bookService")
public class BookServiceImpl implements BookService{
    @Autowired
    private BookRepository repository;

    @Override
    public List<Book> findBookAll() {
        return (List<Book>) repository.findAll();
    }

    @Override
    public void delete(Long id) {
        repository.deleteById(id);
    }

    @Override
    public Book findBookByName(String name) {
        return repository.findBookByName(name);
    }

    @Override
    public Book findBookByCategory(String category) {
        return repository.findBookByCategory(category);
    }

    @Override
    public Book save(Book book) {
        return repository.save(book);
    }
}
````

#### RestController생성

````java
package com.jsh.jpaExample.controller;

import com.jsh.jpaExample.entity.Book;
import com.jsh.jpaExample.service.BookService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@Slf4j
public class BookController {
    @Autowired
    private BookService service;

    @GetMapping("/book")
    public List<Book> findAllBook(){
        List<Book> books    = service.findBookAll();
        books.forEach(System.out::println);
        return books;
    }

    @PostMapping("/book")
    public Book save(@RequestBody Book book){
        Book resBook    = service.save(book);
        log.info(resBook.toString());
        return resBook;
    }

    @PutMapping("/book/{bookId}")
    public Book updateBook(@PathVariable("bookId") Long bookId, @RequestBody Book book){
        book.setId(bookId);
        Book resBook    = service.save(book);
        log.info(resBook.toString());
        return resBook;
    }

    @DeleteMapping("/book/{bookId}")
    public void deleteBook(@PathVariable("bookId") Long bookId){
        service.delete(bookId);
    }
}
````

#### POST
> curl POST http://localhost:8081/book -h '{Content-Type: application/json}' -d '{"name": "Korean Book",
"category": "1"}'

```groovy
# Hibernate Query
Hibernate:
/* insert com.jsh.jpaExample.entity.Book
    */ insert
into
book
(category, created_at, name, sell_count, updated_at)
        values
(?, ?, ?, ?, ?)

# Insert 한 데이터
11:37 INFO  c.j.j.controller.BookController - Book{id=3, name='Korean Book', category='1', sellCount=0, createdAt=2021-04-19T11:37:50.820, updatedAt=2021-04-19T11:37:50.820}
```

#### GET
> curl GET http://localhost:8081/book

````groovy
# Hibernate Query
Hibernate: 
    /* select
        generatedAlias0 
    from
        Book as generatedAlias0 */ select
            book0_.id as id1_0_,
            book0_.category as category2_0_,
            book0_.created_at as created_3_0_,
            book0_.name as name4_0_,
            book0_.sell_count as sell_cou5_0_,
            book0_.updated_at as updated_6_0_ 
        from
            book book0_

# SELECT 된 데이터
Book{id=3, name='Korean Book', category='1', sellCount=0, createdAt=2021-04-19T11:37:50.820, updatedAt=2021-04-19T11:37:50.820}
````

#### PUT

> curl PUT http://localhost:8081/book -h '{Content-Type: application/json}' -d '{
"name": "Korean Book2",
"category": "1"
}'
 

```groovy
# Hibernate Query
Hibernate: 
    /* load com.jsh.jpaExample.entity.Book */ select
        book0_.id as id1_0_0_,
        book0_.category as category2_0_0_,
        book0_.created_at as created_3_0_0_,
        book0_.name as name4_0_0_,
        book0_.sell_count as sell_cou5_0_0_,
        book0_.updated_at as updated_6_0_0_ 
    from
        book book0_ 
    where
        book0_.id=?
Hibernate: 
    /* update
        com.jsh.jpaExample.entity.Book */ update
            book 
        set
            category=?,
            name=?,
            sell_count=?,
            updated_at=? 
        where
            id=?

# Update 된 데이터                    
13:03 INFO  c.j.j.controller.BookController - Book{id=3, name='Korean Book2', category='1', sellCount=0, createdAt=null, updatedAt=2021-04-19T13:03:11.224}
```

#### DELETE
> curl DELETE http://localhost:8081/book -h '{Content-Type: application/json}' -d '{
"name": "Korean Book2",
"category": "1"
}'

```groovy
# Hibernate Query
Hibernate: 
    select
        book0_.id as id1_0_0_,
        book0_.category as category2_0_0_,
        book0_.created_at as created_3_0_0_,
        book0_.name as name4_0_0_,
        book0_.sell_count as sell_cou5_0_0_,
        book0_.updated_at as updated_6_0_0_ 
    from
        book book0_ 
    where
        book0_.id=?
Hibernate: 
    /* delete com.jsh.jpaExample.entity.Book */ delete 
        from
            book 
        where
            id=?
```

github : [https://github.com/suhojang/SpringJPA](https://github.com/suhojang/SpringJPA)
