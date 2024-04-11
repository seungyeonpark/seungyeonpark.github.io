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
last_modified_at: 2024-04-11
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

### 4-1. SADD
```
SADD myset A
SADD myset A A A C B D D E F F F F G
```
- SADD 커맨드를 사용해 set에 아이템을 저장할 수 있으며 한 번에 여러 개의 아이템 저장 가능
- SADD 커맨드는 저장되는 실제 아이템 수를 반환

### 4-2. SMEMBERS
- set에 저장된 전체 아이템을 출력
- 데이터를 저장한 순서와 관계없이 랜덤한 순서로 데이터 출력

### 4-3. SREM
- set에서 원하는 데이터를 삭제

### 4-4. SPOP
- set 내부의 아이템 중 랜덤으로 하나의 아이템을 반환하는 동시에 set에서 그 아이템을 삭제

### 4-5. SUNION/SINTER/SDIFF
- 합집합은 SUNION, 교집합은 SINTER, 차집합은 SDIFF

<br>
<br>

## 5. Sorted set
- sorted set은 스코어 값에 따라 정렬되는 고유한 문자열의 집합
- 모든 아이템은 스코어-값 쌍을 가지며, 저장될 때부터 스코어 값으로 정렬돼 저장된다
- 같은 스코어를 가진 아이템은 데이터의 사전 순으로 정렬돼 저장
- 데이터가 중복 없이 유일하게 저장된다는 점에서 set과 유사
- 각 아이템은 스코어라는 데이터에 연결돼 있다는 점에서 hash와 유사
- 또한 모든 아이템은 스코어 순으로 정렬돼 있어, list처럼 인덱스를 이용해 각 아이템에 접근 가능
- 인덱스를 이용해 아이템에 접근할 일이 많다면 list가 아닌 sorted set을 사용하는 것이 더 효율적이다. list에서 인덱스를 이용해 데이터에 접근하는 것은 O(n)으로 처리되지만, sorted set에서는 O(log(n)으로 처리되기 때문이다.

### 5-1. ZADD
```
ZADD score:220817 100 user:B
ZADD score:228017 150 user:A 150 user:C 200 user:F 300 user:E
```
- ZADD 커맨드를 사용하면 sorted set에 아이템을 저장할 수 있으며, 스코어-값 쌍으로 입력해야 한다
- 한 번에 여러 아이템을 입력할 수 있으며, 각 아이템은 sorted set에 저장되는 동시에 스코어 값으로 정렬된다
- 만약 저장하고자 하는 데이터가 이미 sorted set에 속해 있다면 스코어만 업데이트 되며, 업데이트된 스코어에 의해 아이템이 재정렬된다
- 지정한 키가 존재하지 않을 때에는 sorted set 자료구조를 새로 생성하며, 키가 이미 존재하지만 sorted set이 아닐 경우에는 오류를 만환한다
- 스코어는 배정밀도 부동소수점 문자열이어야 한다
- option
  - XX: 아이템이 이미 존재할 때만 스코어를 업데이트한다
  - NX: 아이템이 존재하지 않을 때만 신규 삽입하며, 기존 아이템의 스코어를 업데이트하지 않는다
  - LT: 업데이트하고자 하는 스코어가 기존 아이템의 스코어보다 작을 때만 업데이트한다. 기존에 아이템이 존재하지 않을 때에는 새로운 데이터를 삽입한다
  - GT: 업데이트하고자 하는 스코어가 기존 아이템의 스코어보다 클 때만 업데이트한다. 기존에 아이템이 존재하지 않을 때에는 새로운 데이터를 삽입한다 

### 5-2. ZRANGE
- ZRANGE 커맨드를 사용하면 sorted set에 저장된 데이터를 조회할 수 있으며, start와 stop 범위를 항상 입력해야 한다
- 인덱스로 데이터 조회
  - ZRANGE 커맨드는 기본적으로 인덱스를 기반으로 데이터를 조회하기 때문에 start와 stop 인자에는 검색하고자 하는 첫 번째와 마지막 인덱스를 전달
  - WITHSCORE 옵션을 사용하면 데이터와 함께 스코어 값이 차례대로 출력
  - REV 옵션을 사용하면 데이터는 역순으로 출력
- 스코어로 데이터 조회
  - ZRANGE에 BYSCORE 옵션을 사용하면 스코어를 이용해 데이터 조회 가능
  - start, stop 인자 값으로는 조회하고자 하는 최소, 최대 스코어를 전달해야 한다
- 사전 순으로 데이터 조회
  - 스코어가 같을 때 BYLEX 옵션을 사용하면 사전식 순서를 이용해 특정 아이템을 조회할 수 있다

<br>
<br>

## 6. Bitmap
- Bitmap은 독자적인 자료 구조는 아니며, string 자료 구조에 bit 연산을 수행할 수 있도록 확장한 형태
- string 자료구조가 binary safe하고 최대 512MB의 값을 저장할 수 있는 구조이기 때문에, 2^32의 비트를 가지고 있는 비트맵 형태라고 볼 수 있다

### 6-1. SETBIT
```
SETBIT mybitmap 2 1
```
- SETBIT 커맨드로 비트 저장

### 6-2. GETBIT
```
GETBIT mybitmap 2
```
- GETBIT 커맨드로 저장된 비트 조회

### 6-3. BITFIELD
```
BITFIELD mybitmap SET u1 6 1 SET u1 10 1 SET u1 14 1
```
- BITFIELD 커맨드로 한 번에 여러 비트를 SET

### 6-4. BITCOUNT
```
BITCOUNT mybitmap
```
- BITCOUNT 커맨드로 1로 설정된 비트의 개수 카운팅

<br>
<br>

## 7. Hyperloglog
- hyperloglog는 집합의 원소 개수인 카디널리티를 추정할 수 있는 자료 구조
- 대량 데이터에서 중복되지 않는 고유한 값을 집계할 때 사용
- set과 같은 데이터 구조에서는 중복을 피하기 위해 저장된 데이터를 모두 기억하고 있으며, 저장되는 데이터가 많아질수록 그만큼 많은 메모리를 사용
- hyperloglog는 입력되는 데이터 그 자체를 저장하지 않고 자체적인 방법으로 데이터를 변경해 처리한다.
  - 저장되는 데이터 개수에 구애받지 않고 계속 일정한 메모리를 유지할 수 있으며, 중복되지 않는 유일한 원소의 개수를 계산할 수 있다
- 하나의 hyperloglog 자료구조는 최대 12KB의 크기를 가지며, 레디스에서 카디널리티 추정의 오차는 0.81%로, 비교적 정확하게 데이터를 추정할 수 있다
- 하나의 hyperloglog 자료구조는 최대 2^64 개의 아이템을 저장할 수 있다
- PFADD 커맨드
  - hyperloglog에 아이템 저장
- PFCOUNT 커맨드
  - 저장된 아이템의 개수, 즉 카디널리티를 추정할 수 있다

<br>
<br>

## 8. Geospatial
- Geospatial 자료구조는 경도, 위도 데이터 쌍의 집합으로 간편하게 지리 데이터를 저장
- 내부적으로 데이터는 sorted set으로 저장되며, 하나의 자료구조 안에 키는 중복 저장되지 않는다
- `GEOADD <key> 경도 위도 member` 순서로 저장
- sorted set과 마찬가지로 XX 옵션을 사용하면 이미 아이템이 있는 경우에만, NX 옵션을 사용하면 아이템이 없는 경우에만 데이터를 저장
- GEOPOS 커맨드
  - 저장된 위치 데이터 조화
- GEODIST 커맨드
  - 두 아이템 사이의 거리 반환
- GEOSEARCH 커맨드
  - 특정 위치를 기준으로 원하는 거리 내의 아이템 검색
  - BYRADIUS 옵션을 사용하면 반경 거리를 기준으로 조회
  - BYBOX 옵션을 사용하면 직사각형 거리를 기준으로 데이터를 조회

<br>
<br>

## 9. Stream
- stream은 레디스를 메시지 브로커로서 사용할 수 있게 하는 자료 구조
- stream 자료구조는 데이터를 계속해서 추가하는 방식(append-only)으로 저장하므로, 실시간 이벤트 혹은 로그성 데이터 저장을 위해 사용 가능