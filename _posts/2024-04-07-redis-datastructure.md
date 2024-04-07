---
title: "Redis 자료구조"
excerpt: "Redis 자료구조에 대해 알아보자"

categories:
  - Redis
tags:
  - [redis]

permalink: /redis/redis=datastructure

toc: true
toc_sticky: true

date: 2024-04-07
last_modified_at: 2024-04-07
---

## 1. String
- String에는 최대 512MB의 문자열 데이터를 저장할 수 있다
- 이진 데이터를 포함하는 모든 종류의 문자열이 binary-safe하게 처리되기 때문에 JPEG 이미지와 같은 바이트 값, HTTP 응답값 등의 다양한 데이터를 저장하는 것도 가능

```
SET hello world
GET hello

SET hello newval NX
SET hello newval XX

SET counter 100
INCR counter
INCRBY counter 50

MSET a 10 b 20 c 30
MGET a b c
```

<br>
<br>

## 2. List

<br>
<br>

## 3. Hash

<br>
<br>

## 4. Set

<br>
<br>

## 5. Sorted set

<br>
<br>

## 6. Bitmap

<br>
<br>

## 7. Hyperloglog

<br>
<br>

## 8. Geospatial

<br>
<br>

## 9. Stream