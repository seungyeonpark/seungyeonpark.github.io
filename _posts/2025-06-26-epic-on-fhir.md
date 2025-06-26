---
title: "SMART on FHIR Standalone Launch 인증 플로우"
description: "SMART on FHIR Standalone Launch 방식에서의 인증 및 데이터 획득"
date: 2025-06-26 11:00:00 +0900
categories: [FHIR, OAuth]
tags: [SMART on FHIR, Standalone Launch, OAuth2.0, Access Token, EHR Integration]
math: false
mermaid: true
---

한국의 의료 시스템은 대체로 병원 중심의 정보 관리 체계를 갖추고 있다. 각 병원은 자체 전자의무기록(EMR)을 통해 환자 정보를 기록하고 보관하며, 의료 데이터는 병원 내부에서 통제된다. 반면, 미국은 FHIR(Fast Healthcare Interoperability Resources)라는 의료 데이터 전송 표준을 만들고, 그 위에 OAuth 2.0 기반 인증 체계인 SMART on FHIR를 도입하면서, 외부 애플리케이션이 EHR 시스템과 연동하고 환자 데이터를 조회할 수 있는 환경을 마련하였다.

기존 회사 솔루션은 국내 병원 설치되는 온프레미스 방식으로, 내부망을 통해 원내 DB에 직접 접속하여 데이터를 가져오는 구조이었다. 하지만 이번 6월 TASCS 연례 학술대회 참석을 계기로, 솔루션을 SMART on FHIR 표준에 맞춰 클라우드 기반 EHR 연동 구조로 확장하게 되었다.

우리 애플리케이션은 의료 기관의 EHR 시스템과 3-legged 인증 방식으로 연동되도록 개발하였다. 사용자는 애플리케이션을 통해 의료 기관의 인증 서버로 연결된다. 해당 인증 서버에서는 로그인 인터페이스를 제공하며, 사용자는 해당 페이지에서 인증을 수행한다. 자격 증명이 완료되면 인증 서버는 앱에서 토큰을 반환하고, 이후 애플리케이션은 토큰을 사용하여 사용자의 의료 데이터에 접근할 수 있다. 이처럼 사용자가 애플리케이션에서 직접 인증하는 방식을 독립 실행형(Standalone Launch)라고 한다.

이외에도 애플리케이션이 의료 기관의 EHR에서 직접 실행되거나, 사용자의 직접적 승인 없이 시스템 간 승인 만으로 데이터를 주고 받는 백엔드 서비스 방식도 있다.

아래에서는 **Standalone Launch** 방식을 기준으로 어떻게 인증이 이루어지고 사용자 사용자 데이터에 접근하는지 살펴보겠다. 본문에 포함된 이미지와 코드 샘플은 Epic의 공식 문서를 참고하였다. [Epic on FHIR](https://fhir.epic.com/Documentation?docId=developerguidelines)

<br>

애플리케이션에서 환자 정보에 대한 접근 권한을 얻기 위해 인증 서버와 사전에 공유해야 하는 정보는 다음과 같다.

- **client_id**: 애플리케이션의 고유 식별자로 인증 서버에서 애플리케이션을 식별할 때 사용된다. EHR 시스템에 앱을 등록하면 발급되며 이후 진행되는 모든 OAuth 플로우에서 필수값으로 사용된다.
- **redirect_uri**: 인증 후 사용자를 다시 애플리케이션으로 리디렉션할 주소이다. 인증 서버는 해당 URI로 요청이 등록된 애플리케이션에서 왔는지 검증하고 토큰 코드 등을 반환한다.
- **credentials**: 사용자 개입 없이 데이터 접근이 필요한 앱에서 사용하는 인증 정보다. 일반적으로 confidential client로 등록된 앱만 사용 가능하며 EHR 시스템에 사전 등록되어 있어야 한다.

<br>
<br>

Standalone Launch 방식은 아래와 같은 프로세스를 통해 애플리케이션과 EHR을 연동한다.

![Standalone Launch](/assets/img/standalone-launch.png)

위 프로세스는 아래와 같이 다섯 단계로 나눠볼 수 있다.

- Step 1: Authorization Code 요청 단계
- Step 2: 인증 서버가 요청을 처리하는 단계
- Step 3: Authorization Code를 Access Token으로 교환
- Step 4: FHIR API를 사용해 환자 데이터에 접근
- Step 5: Refresh Token으로 새로운 Access Token 획득

<br>
<br>

# Step 1: Authorization Code 요청 단계

애플리케이션은 사용자를 인증하고 권한을 받기 위해 브라우저를 아래 URL로 리디렉션한다.

```
GET https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize?
response_type=code
&redirect_uri=[redirect_uri]
&client_id=[client_id]
&state=[state]
&aud=[audience]
&scope=[scope]
```

요청 파라미터별 의미는 다음과 같다.

- response_type: 이 단계에서는 항상 “code” 고정값을 보내야 한다. OAuth 2.0에서 authorization code grant 방식임을 나타낸다.
- client_id: 애플리케이션 ID이다. EHR에서 애플리케이션을 식별하는데 사용한다.
- redirect_uri: 인증이 완료된 후, 사용자를 리디렉트 할 애플리케이션의 주소다. 이 주소로 인증 결과(authorization code)를 전달하며 애플리케이션 등록 시 사전에 등록되어 있어야 한다.
- state: CSRF 공격을 막기 위해 사용하는 보안용 랜덤 문자열이다. 애플리케이션이 요청할 때 생성되며 인증 서버가 그대로 되돌려주면 애플리케이션은 이후 일치 여부를 확인한다.
- scope: 애플리케이션이 요청하는 데이터 접근 범위이다.
- aud: FHIR 서버의 Base URL이다. 어떤 FHIR 리소스 서버에 접근할 것인지를 명시한다.

애플리케이션이 Confidential Client로 등록되어 있다면 아래 값을 보내지 않아도 된다. 하지만 Public Client(PKCE) 방식으로 등록되었다면 아래 파라미터도 함께 보내야 한다.

- code_challenge: 클라이언트에서 생성한 암호화 토큰 코드로 후속 토큰 요청에서 code_verifier와 짝을 이룬다. 암호화 방식은 SHA-256을 사용한다.
- code_challenge_method: code_challenge의 암호화 방식으로 Epic은 현재 SHA-256만 지원한다. 값은 “S256”으로 고정된다.

# Step 2: 인증 서버가 요청을 처리하는 단계

사용자는 Step 1에서 생성된 로그인 페이지에 접속하여 로그인을 한다. 인증 서버는 사용자의 자격 증명(아이디, 비밀번호)을 확인하고 애플리케이션이 요청한 접근 권한(scope)를 검증한다. 인증 서버에서 인증/권한 요청을 승인하면 사용자의 브라우저는 앱에서 등록한 redirect_uri로 리디렉션 된다.

브라우저가 리디렉션 되는 URL 형태는 아래와 같다.

```
https://[redirect_uri]?
code=[authorization_code]
&state=[original_state]
```

code는 인증 서버에서 생성한 authorization code이다. 애플리케이션은 이 코드를 다음 단계에서 access token으로 교환한다. state는 Step 1에서 애플리케이션이 보낸 state값과 동일한 값이다. 애플리케이션은 이 값을 검증하여 CSRF 공격을 방지하고 요청 무결성을 확인해야 한다.

# Step 3: Authorization Code를 Access Token으로 교환

애플리케이션은 access token을 발급받기 위해 Step 2에서 전달받은 authorization code를 가지고 토큰 엔드포인트에 요청을 보내야 한다.

요청 방식은 HTTP POST이고 Content-Type은 application/x-www-form-urlencoded이다.

요청 파라미터는 애플리케이션이 Public Client로 등록되어 있는지 Confidential Client로 등록되어 있는지에 따라 다르다. Confidential Client 안에서도 client secret을 사용하는지, JWT를 사용하는지에 따라 달라진다.

Public Client의 경우 다음과 같이 요청을 보낼 수 있다.

```
POST https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=yfNg-rSc1t5O2p6jVAZLyY00uOOte5KM1y3YUxqsJQnBKEMNsYqOPTyVqcCH3YXaPkLztO9Rvf7bhLqQTwALHcHN6raxpTbR1eVgV2QyLA_4K0HrJO92et3qRXiXPkj7
&redirect_uri=https%3A%2F%2Ffhir.epic.com%2Ftest%2Fsmart
&client_id=d45049c3-3441-40ef-ab4d-b9cd86a17225 
```

여기서 grant_type은 “authorization_code”로 고정된 값을 보낸다. code는 Step 2에서 받은 authorization code를 값으로 보내면 된다. redirect_uri는 Step 1에서 사용한 redirect URI 값을 설정한다. client_id는 인증 서버에서 발급한 애플리케이션 ID를 설정한다.

PKCE를 사용하는 경우 code_verifier를 추가로 보내야 한다. 해당 값은 클라이언트가 생성한 verifier 문자열이다.

Confidential Client이고 client secret 값을 사용하는 경우는 아래와 같이 요청을 보낸다.

```
POST https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded 
Authorization: Basic ZDQ1MDQ5YzMtMzQ0MS00MGVmLWFiNGQtYjljZDg2YTE3MjI1OnRoaXMtaXMtdGhlLXNlY3JldC0yJTJGNw==

grant_type=authorization_code
&code=yfNg-rSc1t5O2p6jVAZLyY00uOOte5KM1y3YUxqsJQnBKEMNsYqOPTyVqcCH3YXaPkLztO9Rvf7bhLqQTwALHcHN6raxpTbR1eVgV2QyLA_4K0HrJO92et3qRXiXPkj7
&redirect_uri=https%3A%2F%2Ffhir.epic.com%2Ftest%2Fsmart
```

client_id, client_secret은 Authorization 헤더에 포함한다.
요청 파라미터는 Public Client에서 사용한 파라미터와 동일하게 보내면 된다.

Confidential Client이고 JWT를 사용하는 경우는 아래와 같이 요청을 보낸다.

```
POST https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&code=yfNg-rSc1t5O2p6jVAZLyY00uOOte5KM1y3YUxqsJQnBKEMNsYqOPTyVqcCH3YXaPkLztO9Rvf7bhLqQTwALHcHN6raxpTbR1eVgV2QyLA_4K0HrJO92et3qRXiXPkj7
&redirect_uri=https%3A%2F%2Ffhir.epic.com%2Ftest%2Fsmart
&client_assertion=eyJhbGciOiJSUzM4NCIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJkNDUwNDljMy0zNDQxLTQwZWYtYWI0ZC1iOWNkODZhMTcyMjUiLCJzdWIiOiJkNDUwNDljMy0zNDQxLTQwZWYtYWI0ZC1iOWNkODZhMTcyMjUiLCJhdWQiOiJodHRwczovL2ZoaXIuZXBpYy5jb20vaW50ZXJjb25uZWN0LWZoaXItb2F1dGgvb2F1dGgyL3Rva2VuIiwianRpIjoiZjllYWFmYmEtMmU0OS0xMWVhLTg4ODAtNWNlMGM1YWVlNjc5IiwiZXhwIjoxNTgzNTI0NDAyLCJuYmYiOjE1ODM1MjQxMDIsImlhdCI6MTU4MzUyNDEwMn0.dztrzHo9RRwNRaB32QxYLaa9CcIMoOePRCbpqsRKgyJmBOGb9acnEZARaCzRDGQrXccAQ9-syuxz5QRMHda0S3VbqM2KX0VRh9GfqG3WJBizp11Lzvc2qiUPr9i9CqjtqiwAbkME40tioIJMC6DKvxxjuS-St5pZbSHR-kjn3ex2iwUJgPbCfv8cJmt19dHctixFR6OG-YB6lFXXpNP8XnL7g85yLOYoQcwofN0k8qK8h4uh8axTPC21fv21mCt50gx59XgKsszysZnMDt8OG_G4gjk_8JnGHwOVkJhqj5oeg_GdmBhQ4UPuxt3YvCOTW9S2vMikNUnxrhdVvn2GVg
```

요청 파라미터 중 grant_type, code, redirect_uri는 Public Client에서 사용한 파라미터와 동일하다.
client_assertion_type은 “urn:ietf:params:oauth:client-assertion-type:jwt-bearer”로 고정된 값을 보내야 한다.
client_assertion은 클라이언트가 서명한 JWT를 설정하면 된다.

응답값의 경우 Public Client은 아래와 같다.

```
{
	"access_token": "Nxfve4q3H9TKs5F5vf6kRYAZqzK7j9LHvrg1Bw7fU_07_FdV9aRzLCI1GxOn20LuO2Ahl5RkRnz-p8u1MeYWqA85T8s4Ce3LcgQqIwsTkI7wezBsMduPw_xkVtLzLU2O",
	"token_type": "bearer",
	"expires_in": 3240,
	"scope": "Patient.read Patient.search ",
	"patient": "T1wI5bk8n1YVgvWk9D05BmRV0Pi3ECImNSK8DKyKltsMB"
}
```

Confidential Client의 경우 아래와 같이 응답값이 리턴된다.

```
{
  "access_token": "Nxfve4q3H9TKs5F5vf6kRYAZqzK7j9LHvrg1Bw7fU_07_FdV9aRzLCI1GxOn20LuO2Ahl5RkRnz-p8u1MeYWqA85T8s4Ce3LcgQqIwsTkI7wezBsMduPw_xkVtLzLU2O",
  "refresh_token": "H9TKs5F5vf6kRYAZqzK7j9L_07_FdV9aRzLCI1GxOn20LuO2Ahl5R1MeYWqA85T8s4sTkI7wezBsMduPw_xkLzLU2O",
  "token_type": "bearer",
  "expires_in": 3240,
  "scope": "Patient.read Patient.search ",
  "patient": "T1wI5bk8n1YVgvWk9D05BmRV0Pi3ECImNSK8DKyKltsMB"
}
```

# Step 4: FHIR API를 사용해 환자 데이터에 접근

Step 3을 통해 유효한 access token을 얻었다면, FHIR API를 사용해 환자 데이터를 조회할 수 있다. 이때 요청에는 Authorization 헤더를 포함해야 하며, Bearer 토큰 형식으로 엑세스 토큰을 전달해야 한다.

요청 예시는 다음과 같다.

```
GET https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/DSTU2/Patient/T1wI5bk8n1YVgvWk9D05BmRV0Pi3ECImNSK8DKyKltsMB HTTP/1.1
Authorization: Bearer Nxfve4q3H9TKs5F5vf6kRYAZqzK7j9LHvrg1Bw7fU_07_FdV9aRzLCI1GxOn20LuO2Ahl5RkRnz-p8u1MeYWqA85T8s4Ce3LcgQqIwsTkI7wezBsMduPw_xkVtLzLU2O
```

# Step 5: Refresh Token으로 새로운 Access Token 획득

기존 access token의 유효 기간이 만료되면, refresh token을 사용해 새 access token을 요청할 수 있다. 이 기능은 애플리케이션이 Confidential Client로 등록된 경우에만 사용할 수 있다.

애플리케이션이 Confidential Client면서 client secret 인증 방식을 사용하는 경우 아래와 같이 요청할 수 있다.

```
POST /oauth2/token
Authorization: Basic base64(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token={이전 refresh token}
```

client_id와 client_secret을 사용해 Basic 인증 헤더를 구성한다.

응답은 다음과 같이 온다.

```
{
  "access_token": "...",
  "token_type": "bearer",
  "expires_in": 3240,
  "scope": "Patient.read Patient.search"
}
```

애플리케이션이 Confidential Client면서 JWT 인증 방식을 사용하는 경우 아래와 같이 요청할 수 있다.

```
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
&client_assertion={JWT}
&refresh_token={이전 refresh token}
```

애플리케이션은 JWT를 생성하여 인증서버에 서명된 방식으로 제출한다.
응답 형식은 client_secret을 사용하는 경우와 동일하다.
