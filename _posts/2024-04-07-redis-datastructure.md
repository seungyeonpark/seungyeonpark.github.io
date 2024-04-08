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

### 1-1. GET
```
GET hello
```
- 데이터 조회

### 1-2. SET
```
SET hello world
```
- `key`: hello, `value`: world
- 이미 키가 존재하는 경우 새로 입력된 값으로 대체된다
- NX 옵션
  ```
  SET hello newval NX
  ```
  - 지정한 키가 없을 때만 새로운 키를 저장
- XX 옵션
  ```
  SET hello newval XX
  ```
  - 키가 이미 있을 때만 새로운 값으로 덮어 쓰며 새로운 키를 생성하진 않는다

### 1-3. INCR/INCRBY
```
INCR counter
INCRBY counter 50
```
- INCR/INCRBY를 사용하면 string 자료 구조에 저장된 숫자를 원자적으로(atomic) 조작할 수 있다
- DECR/DECRBY 커맨드는 동일한 방식으로 데이터를 감소시키는 커맨드이다
- 커맨드가 원자적이라는 건 같은 키에 접근하는 여러 클라이언트가 경쟁 상태(race condition)를 발생시킬 일이 없음을 의미한다
- 

### 1-4. MSET/MGET
```
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