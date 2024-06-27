---
title: "app_store_scraper 라이브러리 사용 시 발생하는 UnicodeEncodeError"
excerpt: ""

categories:
  - ETC
tags:
  - [app_store_scraper]

permalink: /etc/app_store_scraper

toc: true
toc_sticky: true

date: 2024-06-27
last_modified_at: 2024-06-27
---

### 1. Issue:
```
UnicodeEncodeError: 'latin-1' codec can't encode characters in position 30-31: ordinal not in range(256)
```
위 에러는 Python 표준 라이브러리의 http.client 모듈의 putheader 메서드가 헤더 값을 latin-1 코덱을 사용하여 인코딩을 시도할 때 발생하였다.
app_store_scraper를 사용하기 위해서는 앱 이름을 전달해야 하는데, 앱 이름이 한글이기 때문에 latin-1로 인코딩 되지 않는다.

### 2. Troubleshooting:
latin-1으로 인코딩할 수 없는 문자를 무시하도록 http.client 모듈을 수정
- as-is
  ```
  values[i] = one_value.encode('latin-1')
  ```
- to-be
  ```
  values[i] = one_value.encode('latin-1', 'ignore')
  ```
