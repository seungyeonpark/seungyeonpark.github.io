---
title: "Logstash JDBC: Output 플러그인 설정 가이드"
description: "JDBC output 플러그인 설치, 드라이버 설정, 연결 설정 파일 구성 방법"
date: 2025-02-19 23:40:00 +0900
categories: [Database, ETL]
tags: [Logstash, JDBC, Plugin, Database Integration, Configuration]
math: false
mermaid: true
---

```
📌 Logstash에서는 input-jdbc 플러그인만 공식적으로 지원! 
   output-jdbc는 별도로 설치해야 사용 가능!
```

## 1. logstash-output-jdbc 플러그인 설치
``` bash
bin/logstash-plugin install logstash-output-jdbc
```

## 2. JDBC 드라이버 설치
- 드라이버 설치 경로
  - 옵션 1: default 경로
    - `$LOGSTASH_HOME/vendor/jar/jdbc` 경로에 jar 파일 위치시킨다.
    - 추가적인 설정은 필요하지 않음
  - 옵션 2: 사용자 지정 경로
    - 원하는 경로에 jar 파일 위치 후 config에서 driver_jar_path 옵션 설정

## 3. logstash.conf
```
input
{
    stdin { }
}
output {
    jdbc {
        connection_string => "jdbc:oracle:thin:USER/PASS@HOST:PORT:SID"
        statement => [ "INSERT INTO log (host, timestamp, message) VALUES(?, CAST (? AS timestamp), ?)", "host", "@timestamp", "message" ]
    }
}
```
- `connection_string`
  - required option!
- `driver_jar_path`
  - JDBC를 사용자 지정 경로에 위치시킨 경우 해당 옵션에 JDBC 드라이버 경로를 명시해줘야 한다