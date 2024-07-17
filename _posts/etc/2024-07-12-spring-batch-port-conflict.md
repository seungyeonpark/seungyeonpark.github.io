---
title: "Spring Batch 실행 중 발생한 Port 충돌 오류"
excerpt: ""

categories:
  - ETC
tags:
  - [Spring-Batch]

permalink: /etc/spring-batch-port-conflict

toc: true
toc_sticky: true

date: 2024-07-12
last_modified_at: 2024-07-12
---

```
📌 Spring Batch로 개발된 서로 다른 배치 애플리케이션 두 개를 스케줄링 없이 수동으로 실행 중 포트 충돌 에러 발생!
```

<br>

일반적인 Spring Batch 애플리케이션의 경우 웹 서버를 필요로하지 않지만, 배치 프로세스 내에서 웹 통신을 위해 spring-boot-starter-web와 같은 의존성을 추가하는 경우가 있다.
이때 spring-boot-starter-web 의존성을 추가하면 내장 톰캣 서버가 실행된다.
따로 포트 설정을 하지 않으면 기본 8080으로 설정되므로 두 개의 프로세스를 동시에 실행했을 때 포트 충돌 에러가 발생할 수 있다. 
혹은 배치를 여러 번 실행 시켰을 때 Tomcat이 제대로 shutdown하지 못해도 동일한 문제가 발생한다.

<br>

Spring Boot Actuator나 Spring Cloud Data Flow와 같은 모니터링 시스템을 사용하지 않는다면 아래와 같이 내장 톰캣 서버를 비활성화하여 해당 문제를 해결할 수 있다.

- application.yml
    ``` yaml
    spring:
      main:
        web-application-type: none
    ```

<br>

위 설정을 추가하면 Spring Boot 자체적으로 해당 애플리케이션을 웹 애플리케이션으로 간주하지 않으며, 내장 톰캣 서버를 시작하지 않는다. 

<br>

추가적으로 spring.main.web-application-type 설정은 Spring Boot 애플리케이션의 웹 애플리케이션 타입을 지정하는데, 아래와 같은 세 가지 값이 존재한다.
- NONE: 웹 서버를 사용하지 않도록 설정
- SERVLET: 서블릿 기반의 웹 애플리케이션 설정(기본값)
- REACTIVE: 리액티브 웹 애플리케이션으로 설정