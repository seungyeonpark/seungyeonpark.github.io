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

### 1-4. MSET/MGET
```
MSET a 10 b 20 c 30
MGET a b c
```
- MSET/MGET 커맨드를 이용해 한 번에 여러 키를 조작할 수 있다

<br>
<br>

## 2. List
- Redis에서 list는 순서를 가지는 문자열의 목록
- 하나의 list에는 최대 42억여 개의 아이템을 저장할 수 있다
- 일반적인 배열처럼 인덱스를 이용해 데이터에 직접 접근할 수도 있으며, 서비스에서는 스택과 큐로서 많이 사용된다

### 2-1. LPUSH/RPUSH
- LPUSH 커맨드는 list의 왼쪽(head)에 데이터를 추가
- RPUSH 커맨드는 list의 오른쪽(tail)에 데이터를 추가

### 2-2. LRANGE
- LRANGE는 시작과 끝 아이템의 인덱스를 각각 인수로 받아 출력

### 2-3. LPOP
- list에 저장된 첫 번째 아이템을 반환하는 동시에 list에서 삭제
- 숫자와 함께 사용하면 지정한 숫자만큼의 아이템을 반복해서 반환

### 2-4. LTRIM
- 시작과 끝 아이템의 인덱스를 인자로 전달받아 지정한 범위에 속하지 않는 아이템은 모두 삭제하지만, LPOP과 같이 삭제되는 아이템을 반환하지는 않는다
- LPUSH와 LTRIM 커맨드를 사용해 고정 길이 큐 유지하기
    ```
    LPUSH logdata <data>
    LTRIM logdata 0 999
    ```
    - list에 데이터를 저장하면서 매번 1000번째 이상의 인덱스를 삭제
    - 데이터를 일단 쌓은 뒤 주기적으로 배치 처리를 이용해 삭제하는 것보다 위와 같은 방식으로 삭제하는 것이 효율적
    - list에서 tail 데이터를 삭제하는 작업은 O(1)으로 동작하지만, 배치를 이용한다면 처리 시마다 삭제할 데이터를 검색하기 때문

### 2-5. LINSERT
- 원하는 데이터의 앞이나 뒤에 데이터 추가 가능
- 데이터의 앞에 추가하려면 BEFORE 옵션을, 뒤에 추가하려면 AFTER 옵션을 추가
- 만약 지정한 데이터가 없는 경우 오류를 반환

### 2-6. LSET
- 지정한 인덱스의 데이터를 신규 입력하는 데이터로 덮어 쓴다
- 만약 list의 범위를 벗어난 인덱스를 입력하면 에러를 반환

### 2-7. LINDEX
- 원하는 인덱스의 데이터 확인

<br>
<br>

## 3. Hash
### 3-1. HSET
```
HSET Product:123 Name "Hello world"
HSET Product:123 TypeID 35
HSET Product:123 Version 2002
HSET Product:234 Name "Track Ball" TypeID 32
```
- HSET 커맨드를 사용해 hash에 아이템을 저장하며, 한 번에 여러 필드-값 쌍을 저장할 수도 있다

### 3-2. HGET
```
HGET Product:123 TypeID
HMGET Product:234 Name TypeID
HGETALL Product:234
```
- HGET 커맨드를 사용해 hash에 저장된 데이터를 가져올 수 있으며, 이때는 hash 자료 구조의 키와 아이템의 필드를 함께 입력해야 한다
- HMGET 커맨드를 이용하면 하나의 hash 내에서 다양한 필드의 값을 가져올 수 있다
- HGETALL 커맨드는 hash 내의 모든 필드-값 쌍을 차례로 반환한다

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