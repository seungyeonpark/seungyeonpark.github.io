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
last_modified_at: 2024-07-01
---

### 1. Issue
```
UnicodeEncodeError: 'latin-1' codec can't encode characters in position 30-31: ordinal not in range(256)
```
app_store_scrapper 모듈은 앱 이름을 URL의 일부로 사용하나, URL 인코딩을 적용하지 않고 있다. <br>
따라서 한글 앱 이름을 입력하면 위와 같은 에러가 발생한다.

### 2. Troubleshooting
app_store_scrapper 모듈의 base.py를 수정하여 URL 인코딩 적용하도록 수정

  ``` python
  import re
  from urllib.parse import quote
  
  class Base:
  
      # 기존 코드 생략
      def __init__(self, country, app_name, app_id=None, ...):
          self.country = str(country).lower()
          self.app_name = quote(re.sub(r"[\W_]+", "-", str(app_name).lower()), safe='')
          if app_id is None:
              logger.info("Searching for app id")
              app_id = self.search_id()
          self.app_id = int(app_id)
  
          self.url = self._landing_url()
          # 나머지 초기화 코드 생략
  
      def _landing_url(self):
          landing_url = f"{self._base_landing_url}/{self._landing_path}"
          return landing_url.format(
              country=self.country, app_name=self.app_name, app_id=self.app_id
          )
  
      def _request_url(self):
          request_url = f"{self._base_request_url}/{self._request_path}"
          return request_url.format(country=self.country, app_id=self.app_id)
  ```
