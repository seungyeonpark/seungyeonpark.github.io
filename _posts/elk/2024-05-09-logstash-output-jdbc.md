---
title: "logstash-output-jdbc 플러그인"
excerpt: "logstash-output-jdbc 플러그인에 대해 알아보자"

categories:
  - ELK
tags:
  - [logstash]

permalink: /elk/logstash/logstash-output-jdbc

toc: true
toc_sticky: true

date: 2024-05-09
last_modified_at: 2024-05-09
---

input-jdbc 플러그인은 기본으로 내장되어 있지만 output-jdbc 플러그인은 별도로 설치해야 사용 가능

1. output-jdbc 플러그인 설치
    bin/logstash-plugin install logstash-output-jdbc

2. jdbc 드라이버 설치
2-1. default 경로
   vendor/jar/jdbc 경로에 jar 파일 위치시킨다. 
   추가적인 설정은 필요하지 않음
2-2. 사용자 지정 경로
   원하는 경로에 jar 파일 위치 후 configuration에서 driver_jar_path 옵션 설정

3. example
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