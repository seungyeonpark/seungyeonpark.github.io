---
title: "P vs NP 문제"
excerpt: "P vs NP 문제에 대해 알아보자"

categories:
  - Algorithm
tags:
  - [p-np-classes]

permalink: /algorithm/p-np-classes

toc: true
toc_sticky: true

date: 2024-04-10
last_modified_at: 2024-04-10
---

## 1. P 분류 (P class)
- 판정 문제들을 분류하는 방법 중 하나
- 결정론적 튜링 기계에서 다항식 시간 안에 풀 수 있는 모든 문제를 포함
- 결정론적 튜링 기계
  - 어떤 명령어 실행 뒤, 다음 실행할 명령어가 확정됨
  - 코어 하나에서 명령어를 순서대로 실행한다 생각할 것
- 코어 하나에서 실행되는 다항식 시간 알고리듬이 있는 문제는 P

## 2. NP 분류 (NP class)
- 비결정적 다항식 시간(Nondeterministic Polynomial Time)
  - Not P가 아님!
- 비결정론적 튜링 기계
  - 어떤 명령어 실행 뒤, 다음 실행할 명령어가 확정되지 않음
  - 여러 개의 다음 명령어를 병렬적으로 실행하는 기계
- 결정론적 튜링 기계에서의 NP 문제 
  - 일단 답이 있으면 그 답이 맞는지를 다항식 시간 안에 검증할 수 있음 
  - 푸는 데는 지수 시간이 걸릴 수도 있음 그래도 다항식 시간 안에 검증 가능
- 모든 P 문제는 NP이다

## 3. NP 완전, NP 난해

## 4. P vs NP 문제