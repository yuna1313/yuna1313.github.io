---
published: true
title: "[Big Data] ElasticSearch (1)"  
categories:
  - Big Data
tags:
  - search engine

toc: true,
toc_sticky: true

date: 2024-04-30
last_modified_at: 2024-04-30
---

## 1.ElasticSearch란?
ElasticSearch는 Apache Lucene 라이브러리 기반으로 하는 검색 엔진이다.<br>
역인덱스 구조로 데이터를 저장하여 데이터 처리가 빠르며 실시간 분석 시스템이라는 특징이 있다. 또한, 쿼리와 클러스터 설정 등 모든 정보를 JSON 형태로 주고 받는다.<br>
흔히 구글, 네이버, 다음, 무신사 등 여러 곳에서 검색 엔진을 사용한다.
<img width="552" alt="image" src="https://github.com/yuna1313/yuna1313.github.io/assets/93983333/80c3e791-fbed-4fb3-bcd2-5350b86136f7">

## 2.Elastic Stack
ElasticSearch를 찾다보면 듣게 되는 단어이다.<br>
ElasticSearch, Logstash, Kibana의 앞글자를 따서 ELK Stack이라고 불리는 기존 아키텍처에 Beats가 추가된 것이다.

### Logstash
Logstash는 데이터 수집 및 전환 도구이다.<br>
실시간 파이프라인 기능을 갖추었으며, 서로 다른 소스의 데이터를 동적으로 통합하고 해당 데이터를 선택한 대상으로 정규화할 수 있다.

### Kibana
Kibana는 데이터를 시각화할 수 있는 도구이다.<br>
데이터를 시각화하여 대시보드를 만들 수 있고, Elastic Stack 클러스터의 상태를 관리, 모니터링 할 수 있다. 또한 데이터를 신속하게 검색하여 분석할 수 있다.

### Beats
Beats는 운영 데이터를 ElasticSearch로 보내기 위해 경량 데이터를 수집하는 에이전트이다.<br>
Kibana에서 시각화하기 전에 데이터를 추가로 처리하고 향상시킬 수 있는 Logstash나 ElasticSearch를 통해 직접 데이터를 보낼 수 있다.
<br>

3가지 모두 ElasticSearch를 사용하면서 유용하게 쓰일 기술이다. 특히 Kibana의 경우 다음에 포스팅할 Spring Boot와 연동하면서 잘 쓰게 되었다.

## 3.구조

|ES|RDBMS|
|:---|:---|
|Index|Database|
|Shard|Partition|
|Type|Table|
|Document|Row|
|Field|Column|
|Mapping|Schema|
|Query DSL|SQL|

앵간한 개발자면 구조는 RDBMS와 비교하는 것이 이해가 더 빠를 것이라 생각하여 RDBMS와 비교하였다.

## 4.REST API

|ES HTTP Method|RDBMS SQL|
|:---|:---|
|GET|SELECT|
|PUT|INSERT|
|POST|UPDATE, SELECT|
|DELETE|DELETE|
|HEAD||

REST API를 통하여 데이터 조작을 지원하는데 이것도 RDBMS의 SQL과 비교하면 좀 더 쉽게 이해할 수 있다.

