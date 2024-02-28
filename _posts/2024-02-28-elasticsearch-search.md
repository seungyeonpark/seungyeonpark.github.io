---
title: "Elasticsearch 검색"
excerpt: "Elasticsearch의 검색에 대해 알아보자"

categories:
  - Elasticsearch
tags:
  - [elasticsearch]

permalink: /elasticsearch/elasticsearch-search

toc: true
toc_sticky: true

date: 2024-02-28
last_modified_at: 2024-02-28
---

## 1. 쿼리 컨텍스트와 필터 컨텍스트
- 쿼리 컨텍스트
  - 질의에 대한 유사도를 계산해 이를 기준으로 더 정확한 결과를 먼저 보여준다
  - 연관성에 따른 스코어 결과를 제공
- 필터 컨텍스트
  - 유사도를 계산하지 않고 일치 여부에 따른 결과만을 반환
  - "예/아니요"의 결과를 제공
  - 스코어 계산을 하지 않아 결과에 대한 업데이트를 수행할 필요가 없기 때문에 캐시를 이용할 수 있다
  - 엘라스틱서치는 기본적으로 힙 메모리의 10%를 캐시에 이용하고 있는데, 캐시를 이용한 빠른 검색을 하려면 필터 컨텍스트를 이용해야 한다

<br>
<br>

## 2. 쿼리 스트링과 쿼리 DSL
- 퀴리 스트링
  - REST API의 URI 주소에 쿼리문을 작성하는 방식으로 실행 가능

  ``` bash
  curl -H "Content-Type: application/x-ndjson" -XGET localhost:9200/kibana_sample_data_ecommerce/_search?q=customer_fullname:Mary
  ```
- 쿼리 DSL
  - REST API의 요청 본문 안에 JSON 형태로 쿼리를 작성
  
  ``` bash
  curl -H "Content-Type: application/x-ndjson" -XGET localhost:9200/kibana_sample_data_ecommerce/_search
  {
    "query" : {
      "match" : {
        "customer_full_name" : "Mary"
      }
    }
  }
  ```

<br>
<br>

## 3. 유사도 스코어
- 쿼리 컨텍스트는 기본적으로 BM25 알고리즘을 이용해 유사도 스코어를 계산
- 유사도 스코어는 질의문과 도큐먼트의 유사도를 표현하는 값으로, 스코어가 높을수록 찾고자 하는 도큐먼트에 가깝다는 사실을 의미
- 검색 시 explain: true를 추가하면 쿼리 내부적인 최적화 방법과 어떤 경로를 통해 검색되었으며 어떤 기준으로 스코어가 계산되었는지 알 수 있다

3-1. 스코어 알고리즘(BM25)
- BM25 알고리즘은 검색, 추천에 많이 사용되는 알고리즘으로 TF(Term Frequency), IDF(Inverse Document Frequency) 개념에 문서 길이를 고려한 알고리즘
- 검색어가 문서에서 얼마나 자주 나타나는지, 검색어가 문서 내에서 중요한 용어인지 등을 판단하는 근거를 제공

3-2. IDF 계산
- 문서 빈도(Document Frequency)는 특정 용어가 '전체 도큐먼트'에서 얼마나 자주 등장했는지를 의미하는 지표
- IDF(Inverse Document Frequency)는 문서 빈도의 역수로 전체 문서에서 자주 발생하는 단어일수록 중요하지 않은 단어로 인식하고 가중치를 낮추는 것

3-3. TF 계산
- 용어 빈도(Term Frequency)는 특정 용어가 '하나의 도큐먼트'에서 얼마나 많이 등장했는지를 의미하는 지표
- 일반적으로 특정 용어가 도큐먼트에서 많이 반복되었다면 그 용어는 도큐먼트의 주제와 연관되어 있을 확률이 높다
- 최종 스코어는 IDF와 TF 그리고 boost 변수(2.2)를 곱하면 된다

<br>
<br>

## 4. 쿼리

4-1. 전문 쿼리와 용어 수준 쿼리
- 전문 쿼리는 전문 검색을 하기 위해 사용되며, 전문 검색을 할 필드는 인덱스 매핑 시 텍스트 타입으로 매핑해야 한다
  - 매치(match) 쿼리, 매치 프레이즈(match phrase) 쿼리, 멀티 매치(multi match) 쿼리, 쿼리 스트링(query string) 쿼리
- 용어 수준 쿼리는 정확히 일치하는 용어를 찾기 위해 사용되며, 인덱스 매핑 시 필드를 키워드 타입으로 매핑해야 한다
  - 일반적으로 숫자, 날짜, 범주형 데이터를 정확하게 검색할 때 사용
  - 용어(term) 쿼리, 용어들(terms) 쿼리, 퍼지(fuzzy) 쿼리

4-2. 매치 쿼리
- 전문 쿼리의 가장 기본이 되는 쿼리
- 전체 텍스트 중에서 특정 용어나 용어들을 검색할 때 사용
- 전문 쿼리의 경우 검색어도 토큰화된다
- 매치 쿼리에서 용어들 간의 공백은 OR로 인식한다

4-3. 매치 프레이즈 쿼리
- 매치 프레이즈 쿼리는 구(phrase)를 검색할 때 사용

4-4. 용어 쿼리
4-5. 용어들 쿼리
4-6. 멀티 매치 쿼리
4-7. 범위 쿼리
4-8. 논리 쿼리
4-9. 패턴 검색