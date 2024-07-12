---
title: "Spring Batch ���� �� �߻��� Port �浹 ����"
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
? Spring Batch�� ���ߵ� ���� �ٸ� ��ġ ���ø����̼� �� ���� �����ٸ� ���� �������� ���� �� ��Ʈ �浹 ���� �߻�!
```

<br>

�Ϲ����� Spring Batch ���ø����̼��� ��� �� ������ �ʿ������ ������, ��ġ ���μ��� ������ �� ����� ���� spring-boot-starter-web�� ���� �������� �߰��ϴ� ��찡 �ִ�.
�̶� spring-boot-starter-web �������� �߰��ϸ� ���� ��Ĺ ������ ����ȴ�.
���� ��Ʈ ������ ���� ������ �⺻ 8080���� �����ǹǷ� �� ���� ���μ����� ���ÿ� �������� �� ��Ʈ �浹 ������ �߻��� �� �ִ�. 
Ȥ�� ��ġ�� ���� �� ���� ������ �� Tomcat�� ����� shutdown���� ���ص� ������ ������ �߻��Ѵ�.

<br>

Spring Boot Actuator�� Spring Cloud Data Flow�� ���� ����͸� �ý����� ������� �ʴ´ٸ� �Ʒ��� ���� ���� ��Ĺ ������ ��Ȱ��ȭ�Ͽ� �ش� ������ �ذ��� �� �ִ�.

- application.yml
    ``` yaml
    spring:
      main:
        web-application-type: none
    ```

<br>

�� ������ �߰��ϸ� Spring Boot ��ü������ �ش� ���ø����̼��� �� ���ø����̼����� �������� ������, ���� ��Ĺ ������ �������� �ʴ´�. 

<br>

�߰������� spring.main.web-application-type ������ Spring Boot ���ø����̼��� �� ���ø����̼� Ÿ���� �����ϴµ�, �Ʒ��� ���� �� ���� ���� �����Ѵ�.
- NONE: �� ������ ������� �ʵ��� ����
- SERVLET: ���� ����� �� ���ø����̼� ����(�⺻��)
- REACTIVE: ����Ƽ�� �� ���ø����̼����� ����