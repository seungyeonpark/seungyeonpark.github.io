---
title: "엘라스틱서치 집계와 통계 분석" 
description: "매트릭/버킷 집계 타입, 파이프라인 집계를 활용한 데이터 분석과 통계"
date: 2025-02-16 23:40:00 +0900
categories: [Database]
tags: [Elasticsearch, Aggregation, Metrics, Bucket, Pipeline]
math: true
mermaid: true
---

## 1. 집계의 요청-응답 형태
- 엘라스틱서치에서 집계는 데이터를 그룹핑하고 통곗값을 얻는 기능으로 SQL의 group by와 통계 함수를 포함
- 집계를 위한 특별한 API가 제공되는 건 아니고 search API 요청 본문에 aggs 파라미터를 이용하면 쿼리 결과에 대한 집계를 생성할 수 있다
- 집계 요청

    ``` json
    {
        "aggs" : {
            "my_aggs" : {
                "agg_type" : {
                    ...
                }
            }
        }
    }
    ```
    - aggs: 집계 요청을 하겠다는 의미
    - my_aggs: 사용자가 지정하는 집계 이름
    - agg_type: 집계 타입 (metric/bucket)
- 집계 응답
  
    ``` json
    {
        "hits" : {
            "total" : {
                ...
            }
        },
        "aggregations" : {
            "my_aggs" : {
                "value" : ...
            }
        }
    }
    ```
    - my_aggs: 집계 요청 시 사용자가 지정한 이름
    - value: 실제 집계 결과

<br>
<br>

## 2. 매트릭 집계
- 매트릭 집계는 필드의 최소/최대/합계/평균/중간값 같은 통계 결과를 보여준다
- 필드의 타입에 따라서 사용 가능한 집계 타입에 제한이 있음
  - 텍스트 타입 필드는 합계나 평균 같은 수치 연산을 계산할 수 없다

<br>

### 2-1. 평균값/중간값 구하기
- 평균값을 구하는 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "stats_aggs" : {
                "avg" : {
                    "field" : "products.base_price"
                }
            }
        }
    }
    ```
    - 평균 집계를 사용하기 위해서는 필드 타입이 정수나 실수 타입이어야 한다
    - "size" : 0을 설정하면 집계에 사용한 도큐먼트를 결과에 포함하지 않음으로써 비용 절약할 수 있음
- 백분위를 구하는 집계 요청
  
    ``` json
    {
        "size" : 0,
        "aggs" : {
            "stats_aggs" : {
                "percentiles" : {
                    "field" : "products.base_price",
                    "percents" : [
                        25,
                        50
                    ]
                }
            }
        }
    }
    ```
    - 백분위 25%, 50%에 속하는 데이터를 요청

<br>

### 2-2. 필드의 유니크한 값 개수 확인하기
- cardinality 집계는 필드의 중복된 값들은 제외하고 유니크한 데이터의 개수만 보여준다(SQL의 distinct count)
- 일반적으로 범주형 데이터에서 유니크한 데이터를 확인하는 용도로 사용
- cardinality 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "cardi_aggs" : {
                "cardinality" : {
                    "field" : "day_of_week",
                    "precision_threshold" : 100
                }
            }
        }
    }
    ```
    - day_of_week 필드의 유니크한 데이터 개수를 요청
    - precision_threshold 파라미터를 이용해 정확도와 리소스를 등가 교환
- precision_threshold는 카디널리티의 실제 결과보다 크게 잡아야 한다. 그렇지 않으면 잘못된 결과를 반환함
  - 하지만 실제 결과를 모르기 때문에 precision_threshold 값을 변경해보며 값이 변경되지 않는 임계점을 찾기도 함
  - 기본 값은 3000이며, 최대 40000까지 값을 설정할 수 있다
- cardinality는 매우 적은 메모리로 집합의 원소 개수를 추정할 수 있는 HyperLogLog++ 알고리즘을 기반으로 동작
  - HyperLogLog++는 집합 내 중복되지 않는 항목의 개수를 세기 위한 알고리즘
  - 완전히 정확한 값을 반환하지는 않으나 일반적으로 5% 이내의 오차를 보이며, 정밀도를 직접 지정해 오차율을 낮출 수 있다
  - cardinality 낮은, 즉 중복을 제거한 항목 수가 적은 집합일수록 100%에 가까운 정확성을 보인다
  - 집계 대상의 크기가 얼마나 크든 간에 지정한 정밀도 이상의 메모리를 사용하지 않기 때문에 대용량 데이터베이스에서 유용
- 버킷 집계의 일종인 용어 집계(terms)를 사용하면 필드의 유니크한 데이터 개수와 데이터 종류를 확인할 수 있다
  - 용어 집계 요청
    
    ``` json
    {
        "size" : 0,
        "aggs" : {
            "cardi_aggs" : {
                "terms" : {
                    "field" : "day_of_week"
                }
            }
        }
    }
    ```

<br>

### 2-3. 검색 결과 내에서의 집계
- 필요한 도큐먼트만 검색한 후 이를 바탕으로 집계
- 쿼리를 이용해 집계 범위 지정

    ``` json
    {
        "size" : 0,
        "query" : {
            "term" : {
                "day_of_week" : "Monday"
            }
        },
        "aggs" : {
            "query_aggs" : {
                "sum" : {
                    "field" : "products.base_price"
                }
            }
        }
    }
    ```
    - term 쿼리를 이용해 day_of_week의 필드값이 "Monday"인 도큐먼트를 일차적으로 골라내고 결과값을 가지고 query_aggs라는 이름으로 products.base_price 필드의 값을 집계

<br>
<br>

## 3. 버킷 집계
- 매트릭 집계가 특정 필드를 기준으로 통곗값을 계산하려면 목적이라면, 버킷 집계는 특정 기준에 맞춰서 도큐먼트를 그룹핑하는 역할
- 버킷은 도큐먼트가 분할되는 단위로 나뉜 각 그룹
- 버킷 집계 종류

    |버킷 집계|설명|
    |---|---|
    |histogram|숫자 타입 필드를 일정 간격으로 분류|
    |date_histogram|날짜/시간 타입 필드를 일정 날짜/시간 간격으로 분류|
    |range|숫자 타입 필드를 사용자가 지정하는 범위 간격으로 분류|
    |date_range|날짜/시간 타입 필드를 사용자가 지정하는 날짜/시간 간격으로 분류|
    |terms|필드에 많이 나타나는 용어(값)들을 기준으로 분류|
    |significant_terms|terms 버킷과 유사하나, 모든 값을 대상으로 하지 않고 인덱스 내 전체 문서 대비 현재 검색 조건에서 통계적으로 유의미한 값들을 기준으로 분류|
    |filters|각 그룹에 포함시킬 문서의 조건을 직접 지정한다. 이때 조건은 일반적으로 검색에 사용되는 쿼리와 동일|

<br>

### 3-1. histogram 집계
- 숫자 타입 필드를 일정 간격 기준으로 구분해주는 집계
- 히스토그램 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "histogram_aggs" : {
                "histogram" : {
                    "field" : "products.base_price",
                    "interval" : 100
                }
            }
        }
    }
    ```
    - histogram_aggs: 집계 이름
    - field: 집계를 진행할 대상 필드명

<br>

### 3-2. range 집계
- 히스토그램 집계는 설정이 간단하지만 각 버킷의 범위를 동일하게 지정할 수밖에 없다
  - 특정 구간에 데이터가 몰려 있거나 데이터 편차가 큰 경우 모든 데이터를 표현하는 데 비효율적
- 범위 집계를 이용하면 각 버킷의 범위를 사용자가 직접 설정할 수 있다
- 범위 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "range_aggs" : {
                "range" : {
                    "field" : "products.base_price",
                    "ranges" : [
                        { "from" : 0, "to" : 50 },
                        { "from" : 50, "to" : 100 },
                        { "from" : 100, "to" : 200 },
                        { "from" : 200, "to" : 1000 }
                    ]
                }
            }
        }
    }
    ```

<br>

### 3-3. terms 집계
- 필드의 유니크한 값을 기준으로 버킷을 나눌 때 사용
- terms 집계 요청

    ``` json
    {
        "size" : 0.
        "aggs" : {
            "term_aggs" : {
                "terms" : {
                    "field" : "day_of_week",
                    "size" : 6
                }
            }
        }
    }
    ```
    - day_of_week 필드의 값을 기준으로 도큐먼트 수가 많은 상위 6개의 버킷을 요청
    - size 파라미터는 만들어지는 버킷 수를 지정하는데, size를 작성하지 않으면 기본값으로 10이 적용
    - size로 지정한 숫자가 생성되는 버킷보다 작은 경우 size로 지정한 버킷만 보이고 나머지 버킷들은 보이지 않는다
- terms 집계 응답
  - doc_count_error_upper_bound: 버킷이 잠재적으로 카운트하지 못할 도큐먼트 수
  - sum_other_doc_count: 버킷에는 있지만 size 때문에 보이지 않는 도큐먼트 수
- terms 집계가 정확하지 않은 이유
  - terms 집계가 부정확도를 표시하는 이유는 분산 시스템의 집계 과정에서 발생하는 잠재적인 오류 가능성 때문
  - 엘라스틱서치는 샤드에 도큐먼트를 저장하고 이를 분산하는데, size 설정값과 샤드 개수 등에 의해 집계에 오류가 발생할 수 있다
- terms 집계 정확성 높이기
  - 고속 처리를 위한 리소스와 속도 간 트레이드오프의 일환으로 리소스 소비량을 늘리면 정확도를 높일 수 있다
  - terms 집계 오류 확인 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "terms_aggs" : {
                "terms" : {
                    "field" : "day_of_week",
                    "size" : 6,
                    "show_term_doc_count_error" : true
                }
            }
        }
    }
    ```
    - show_term_doc_count_error 파라미터를 추가하면 doc_count_error_upper_bound 값을 버킷마다 확인할 수 있다
    - 각 버킷마다 doc_count_error_upper_bound 값이 나오고 0이 나오면 오류가 없다는 뜻
    - 확인 결과 이상값이 나올 경우에는 이를 해결하기 위해 샤드 크기 파라미터를 늘릴 수 있다
- terms 집계 시 샤드 크기를 늘린 요청

  ``` json
  {
      "size" : 0,
      "aggs" : {
          "term_aggs" : {
              "term" : {
                  "field" : "day_of_week",
                  "size" : 6,
                  "shard_size" : 100
              }
          }
      }
  }
  ```
  - 용어 집계 시 shard_size 파라미터를 이용해 샤드 크기를 늘릴 수 있는데 샤드 크기는 용어 집계 과정에서 개별 샤드에서 집계를 위해 처리하는 개수를 의미
  - 샤드 크기를 크게 하면 정확도가 올라가는 대신 리소스 사용량이 올라가 성능은 떨어질 수 있다
  - 샤드 크기는 기본적으로 size * 1.5 + 10으로 계산되는데, 여기서 size는 버킷의 개수다

<br>
<br>

## 4. 집계의 조합
- 관계형 데이터베이스에서 GROUP BY로 그룹핑한 다음 통계 함수를 사용하는 것처럼 버킷 집계와 매트릭 집계를 조합하면 다양한 그룹별 통계를 계산할 수 있다

<br>

### 4-1. 버킷 집계와 매트릭 집계
- 집계의 가장 기본적인 형태는 버킷 집계로 도큐먼트를 그룹핑한 후 각 버킷 집계별 매트릭 집계를 사용하는 것이다
- 버킷 집계 후 매트릭 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "term_aggs" : {
                "terms" : {
                    "field" : "day_of_week",
                    "size" : 5
                },
                "aggs" : {
                    "avg_aggs" : {
                        "avg" : {
                            "field" : "products.base_price"
                        }
                    }
                }
            }
        }
    }
    ```
    - term_aggs라는 용어 집계를 사용해 day_of_week 별로 버킷을 나누는데 상위 5개의 버킷만 사용
    - 매트릭 집계인 평균 집계(avg_aggs)는 용어 집계(term_aggs) 내부에서 호출된다. 
    - 즉 용어 집계로 상위 5개 버킷을 만들고 각각의 버킷 내부에서 products.base_price 필드의 평균값을 구한다

<br>

### 4-2. 서브 버킷 집계
- 버킷 집계로 버킷을 생성한 다음 버킷 내부에서 다시 버킷 집계 (트리 구조)
- 서브 버킷 생성 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "histogram_aggs" : {
                "histogram" : {
                    "field" : "product.base_price",
                    "interval" : 100
                },
                "aggs" : {
                    "term_aggs" : {
                        "terms" : {
                            "field" : "day_of_week",
                            "size" : 2
                        }
                    }
                }
            }
        }
    }
    ```
    - 버킷 집계인 히스토그램 집계를 사용해 products.base_price 필드를 100단위로 구분
    - 두 번째 버킷 집계인 terms 집계는 히스토그램 집계 내부에 있다. 이 경우 히스토그램 집계로 나누어진 버킷 내부에서 다시 버킷을 나눈다
    - day_of_week 필드를 유니크한 값 기준으로 구분하는데 상위 2개의 버킷만 만든다
- 서브 버킷은 2단계를 초과해서 만들지 않는 편이 좋다. 
  - 서브 버킷을 많이 만들수록 버킷의 수는 기하급수적으로 늘어날 수 있으니 주의
  - 집계의 성능이 느려질 뿐만 아니라 클러스터에도 과도한 부하를 가할 수 있다

<br>
<br>

## 5. 파이프라인 집계
- 이전 집계로 만들어진 결과를 입력으로 삼아 다시 집계하는 방식
- 단독으로 사용할 수 없고 반드시 먼저 다른 집계가 있어야 한다
- 파이프라인 집계에는 부모 집계와 형제 집계가 있다
  - 차이점은 집계가 작성되는 위치
  - 부모 집계는 기존 집계 내부에서 작성
  - 형제 집계는 기존 집계 외부에서 작성
- 파이프라인 집계 종류

    |형제/부모 집계|집계 종류|설명|
    |---|---|---|
    |형제 집계|min_bucket|기존 집계 중 최솟값을 구한다|
    |  |max_bucket|기존 집계 중 최댓값을 구한다|
    |  |avg_bucket|기존 집계의 평균값을 구한다|
    |  |sum_bucket|기존 집계의 총합을 구한다|
    |  |stat_bucket|기존 집계의 min, max, sum, count, avg를 구한다|
    |  |percentile_bucket|기존 집계의 백분윗값을 구한다|
    |  |moving_avg|기존 집계의 이동 평균을 구한다. 단, 기존 집계는 순차적인 데이터 구조여야 한다|
    |부모 집계|derivative|기존 집계의 미분을 구한다|
    |  |cumulative_sum|기존 집계의 누적합을 구한다|

<br>

### 5-1. 부모 집계
- 기존 집계 결과를 이용해 새로운 집계를 생성
- 결과는 기존 집계 내부에서 나온다
- 누적합을 구하는 부모 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "histogram_aggs" : {
                "histogram" : {
                    "field" : "products.base_price",
                    "interval" : 100
                },
                "aggs" : {
                    "sum_aggs" : {
                        "sum" : {
                            "field" : "taxful_total_price"
                        }
                    },
                    "cum_sum" : {
                        "cumulative_sum" : {
                            "buckets_path" : "sum_aggs"
                        }
                    }
                }
            }
        }
    }
    ```
    - 부모 집계는 합계 집계(sum_aggs)를 입력으로 받아 최종적으로 각 버킷의 누적합(cumulative_sum)을 계산
    - 파이프라인 집계는 반드시 버킷 경로(buckets_path)를 입력해야 하는데, 입력으로 사용했던 sum_aggs 집계를 적으면 된다

<br>

### 5-2. 형제 집계
- 기존 집계를 참고해 집계를 수행
- 결과는 기존 집계와 동일선상에서 나온다
- 총합을 구하는 형제 집계 요청

    ``` json
    {
        "size" : 0,
        "aggs" : {
            "term_aggs" : {
                "terms" : {
                    "field" : "day_of_week",
                    "size" : 2
                },
                "aggs" : {
                    "sum_aggs" : {
                        "sum" : {
                            "field" : "products.base_price"
                        }
                    }
                }
            },
            "sum_total_price" : {
                "sum_bucket" : {
                    "buckets_path" : "term_aggs>sum_aggs"
                }
            }
        }
    }
    ```
    - 파이프라인 집계는 버킷 병로(buckets_path)를 입력해야 하는데, 누적합과 다르개 > 기호가 들어간다
    - 버킷 경로에서 '>'는 하위 집계 경로를 나타낼 때 사용된다  