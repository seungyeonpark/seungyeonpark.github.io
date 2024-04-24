---
title: "Docker 레지스트리 이미지 공유"
excerpt: "Docker 레지스트리 이미지 공유에 대해 알아보자"

categories:
  - Docker
tags:
  - [docker]

permalink: /docker/docker-registry

toc: true
toc_sticky: true

date: 2024-04-24
last_modified_at: 2024-04-24
---

## 레지스트리, 리포지터리, 이미지 태그
- 로컬 컴퓨터에 없는 이미지를 내려받으려 할 때 가장 먼저 찾아보는 곳이 도커 허브
- 도커 이미지 참조
  ```
  docker.io/diamol/golang:latest
  ```
  - docker.io: 이미지가 저장된 레지스트리의 도메인. 기본값은 도커 허브
  - diamol: 이미지 작성자의 계정 이름. 개인 혹은 단체의 이름에 해당
  - golang: 이미지 레포지토리 이름. 일반적으로 애플리케이션의 이름에 해당한다. 하나의 레포지토리는 여러 버전의 이미지를 담을 수 있다
  - latest: 이미지 태그. 애플리케이션의 버전 혹은 변종을 나타낸다. 기본값은 latest
- 이미지 참조가 레지스트리에서 특정한 이미지를 식별하는 식별자 역할을 한다
  - 이미지는 여러 개의 참조를 가질 수 있다
- 이미지 참조 목록 확인
  ```
  docker image ls --filter reference=image-gallery --filter reference='*/image-gallery'
  ```
  - 출력 결과 리스트에서 이미지 ID가 같다면 두 이미지 참조가 같은 이미지를 가리키는 것이다
- 레지스트리의 기본값은 도커 허브
  - 자사 도커 레지스트리를 꾸리는 경우 이미지 참조의 첫 부분에 인하우스 레지스트리의 도메인을 기재하면 도커는 도커 허브 대신 해당 레지스트리를 찾아간다
- diamol는 공개 레퍼지토리이므로 누구든지 이미지를 내려받을 수 있지만 diamol 단체의 소속원만이 레퍼지토리에 이미지를 푸시할 수 있다
- 태그의 기본값은 latest
  - 레지스트리에 이미지를 푸시할 때는 항상 명시적으로 태그를 부여해야 한다

<br>
<br>

## 도커 허브에 직접 빌드한 이미지 푸시
- 레지스트리에 이미지 푸시
  (1) 도커 CLI를 통해 레지스트리에 로그인. 이 로그인을 통해 레지스트리에 이미지를 푸시할 권한이 부여
  (2) 이미지에 푸시 권한을 가진 계정명을 포함하는 이미지 참조 붙이기
  (3) 로컬 이미지 레이어를 레지스트리로 푸시
- 도커 허브 로그인
  ```
  docker login --username $dockerId
  ```
  - 도커 허브는 기본 레지스트리이므로 도메인 이름을 지정할 필요가 없다
- 이미지 참조 부여
  ```
  docker image tag image-gallery $dockerId/image-gallery:v1
  ```
- 로컬 컴퓨터에 저장된 이미지 레이어를 원격 레지스트리에 푸시
  ```
  docker image push $dockerId/image-gallery:v1
  ```

<br>
<br>

## 나만의 도커 레지스트리 운영

<br>
<br>

## 이미지 태그를 효율적으로 사용

<br>
<br>

## 공식 이미지에서 골든 이미지로 전환