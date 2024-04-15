---
title: "Redis sorted set 활용"
excerpt: "Redis에서 sorted set을 활용하는 방법에 대해 알아보자"

categories:
  - Redis
tags:
  - [redis]

permalink: /redis/redis-sorted-set

toc: true
toc_sticky: true

date: 2024-04-13
last_modified_at: 2024-04-15
---

## 1. 데이터 저장
- 레디스의 sorted set에서 데이터는 저장될 때부터 정렬돼 들어간다. 만약 아이템의 스코어를 sorted set의 가중치로 설정한다면 스코어 순으로 아이템이 정렬되기 때문에 리더보드의 데이터를 읽어오기 위해 매번 데이터를 정렬할 필요가 없다
- sorted set에 데이터를 저장할 때는 ZADD 커맨드를 사용한다
  ```
  ZADD daily-score:240413 28 player:286
  ZADD daily-score:240413 400 player:234
  ZADD daily-score:240413 45 player:101
  ZADD daily-score:240413 357 player:24
  ZADD daily-score:240413 199 player:143
  ```
- 순서 없이 데이터를 저장하더라도 sorted set에는 데이터가 스코어 순으로 정렬돼 저장된다
- ZRANGE 커맨드를 이용하면 스코어로 오름차순 정렬된 데이터를 확인할 수 있다
  ```
  ZRANGE daily-score:240413 0 -1 withscores
  ```
  - ZRANGE는 스코어가 낮은 순서부터 출력한다
- ZREVRANGE는 sorted set에 저장된 데이터를 내림차순으로 반환
  ```
  ZREVRANGE daily-score:240413 0 2 withscores
  ```
  - 0번 인덱스인 첫 번째 데이터부터 2번 인덱스인 세 번째 데이터까지 출력하는 커맨드를 사용했기 때문에 스코어가 가장 높은 3개의 데이터가 출력된다

## 2. 데이터 업데이트
- sorted set은 기본적으로 set이기 때문에 데이터는 중복으로 저장되지 않으며, 같은 아이템을 저장하고자 할 때 스코어가 다르면 기존 데이터의 스코어만 신규 입력한 스코어로 업데이트된다
- 스코어가 업데이트되면 그에 맞춰 데이터의 순서도 다시 정렬된다
- 직접 스코어의 값을 지정해서 변경하지 않고도 ZINCRBY 커맨드를 이용해서 sorted set 내의 스코어를 증감시킬 수 있다.
- ZINCRBY 커맨드는 string에서의 INCRBY 커맨드와 비슷하게 동작하며, 아이템의 스코어를 입력한 만큼 증가시키는 커맨드다
  ```
  ZINCRBY daily-score:240413 100 player:24
  ```
  - 기존 스코어가 357이었던 아이템 player:24의 스코어를 100 증가시키면 아이템은 457로 변경되며 재정렬된다

## 3. 랭킹 합산
- 관계형 데이터베이스에서 주간 누적 랭킹을 구현하려면 하나의 테이블에서 일자에 해당하는 데이터를 모두 가져온 뒤 선수별로 합치고, 이를 다시 sorting하는 작업을 진행해야 함
- 레디스에서는 ZUNIONSTORE 커맨드를 사용해 간단하게 구현할 수 있다
- ZUNIONSTORE 커맨드는 지정한 키에 연결된 각 아이템의 스코어를 합산하는 커맨드
  ```
  ZUNIONSOTRE weekly-score:2404-3 3 daily-score:240413 daily-score:240414 daily-score:240415
  ```
  - 해당하는 일자의 키를 지정하기만 한다면 손쉽게 주간 누적 랭킹을 구현할 수 있다
- sorted set은 스코어가 같을 때는 사전 순으로 정렬된다
  ```
  ZUNIONSOTRE weekly-score:2404-3 3 daily-score:240413 daily-score:240414 daily-score:240415 weights 1 2 1
  ```
  - ZUNIONSTORE를 이용해 데이터베이스를 합칠 때 스코어에 가중치를 둘 수도 있다

## 4. 최신 순 정렬
- 데이터를 저장할 때 유저가 검색한 시간을 스코어로 지정한다면 검색 시간 순으로 정렬된 데이터가 저장된다
  ```
  ZADD search-keyworld:123 20240413143400 코듀로이
  ```
- ZREVRANGE 커맨드를 사용해 가장 최근에 검색한 순서대로 데이터를 가져올 수 있으며, 이때 인덱스를 지정해 최근 5개의 데이터만 조회할 수 있다
  ```
  ZREVRANGE search-keyword:123 0 4 withscores
  ```
- sorted set을 이용해 데이터를 저장하기 때문에 같은 키워드를 다시 검색했을 때도 별다른 중복 체크를 진행하지 않아도 된다
- 각 아이템은 중복되지 않게 저장되기 때문에 같은 아이템의 데이터를 입력한다면 자동으로 스코어만 업데이트돼 재정렬된다

## 5. 아이템을 일정 개수로 유지
- 데이터가 6개째 저장됐을 때 가장 오래된 데이터인 0번 인덱스의 데이터를 삭제해야 한다
- 데이터가 6개 저장됐을 때 가장 오래전 저장된 데이터는 일반 인덱스로 0, 음수 인덱스로는 -6이 된다. 따라서 -6번 인덱스를 삭제하는 것은 0번 인덱스를 삭제하는 것과 동일한 작업을 하는 것이 된다.
  ```
  ZADD search-keyword 20240413144100 버킷햇
  ZREMRANGEBYRANK search-keyword -6 -6
  ```
  - 항상 ZADD로 데이터를 저장할 때마다 음수 인덱스 -6번째를 삭제하는 로직을 추가하면 유저별 아이템의 개수를 확인하지 않더라도 5개 이상의 데이터가 저장되지 않도록 강제할 수 있다
  - ZREMRANGEBYRANK <key> <start> <stop> 커맨드를 이용하면 인덱스의 범위로 아이템을 삭제할 수 있다
  - 만약 아이템의 개수가 5개보다 많지 않을 때에는 -6번째 인덱스는 존재하지 않기 때문에 삭제된 데이터가 없으므로 영향을 주지 않는다
