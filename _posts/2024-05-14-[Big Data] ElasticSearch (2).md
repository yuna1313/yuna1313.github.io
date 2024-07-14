---
published: true
title: "[Big Data] ElasticSearch (2)"  
categories:
  - Big Data
tags:
  - search engine
  - nori

toc: true,
toc_sticky: true

date: 2024-05-14
last_modified_at: 2024-05-14
---

이번엔 저번 글과 이어서 Elastic Search를 설치 및 nori 세팅하는 방법을 알아보려 한다.

## 1.ElasticSearch 설치
아래 사이트를 접속하여 Window, Mac 모두 ElasticSearch를 다운로드 받을 수 있다. <br>
[https://www.elastic.co/kr/downloads/elasticsearch](https://www.elastic.co/kr/downloads/elasticsearch) <br>
<img width="1267" alt="스크린샷 2024-05-14 223738" src="https://github.com/yuna1313/yuna1313.github.io/assets/93983333/824b4e79-0796-4c60-91d9-12333b39d1c8">

설치 후 C:\elasticsearch-8.13.2\bin\elastic.bat 프로젝트를 실행해준다. (C 드라이브에 설치해줘서 그런 것! 다른 경로일 경우 참고)<br>
나의 경우 8.x.x 이상 버전이라 바로 실행이 안 되고 오류가 났다
```
Received plaintext http traffic on an https channel, closing connection
```
그래서 구글링 해봤는데 C:\elasticsearch-8.13.2\config\elasticsearch.yml 파일에서 아래 줄을 주석 쳐주면 정상적으로 작동한다.
```
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
xpack.security.http.ssl: enabled: false
xpack.security.transport.ssl: enabled: false
```
자바 경로가 안 맞는 경우가 있다고 하니 해당 내용도 참고하면 좋을 듯
<img width="815" alt="스크린샷 2024-05-14 225356" src="https://github.com/yuna1313/yuna1313.github.io/assets/93983333/aef145aa-56ad-483b-9ee2-ad90d725f05b">

잘 실행될 경우 실행하면 이런식으로 출력된다

localhost:9200으로 접속하였을 때
```
{
  "name": "",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "{UUID}",
  "version": {
    "number": "8.13.2",
    "build_flavor": "default",
    "build_type": "zip",
    "build_hash": "{HASH}",
    "build_date": "2024-04-05T14:45:26.420424304Z",
    "build_snapshot": false,
    "lucene_version": "9.10.0",
    "minimum_wire_compatibility_version": "7.17.0",
    "minimum_index_compatibility_version": "7.0.0"
  },
  "tagline": "You Know, for Search"
}
```

이런 식으로 나오면 성공


## 2.nori 한글 지원
nori는 ElasticSearch에서 제공하는 한글 형태소 분석기이다.

C:\elasticsearch-8.13.2\bin에서 cmd를 실행시켜 아래 명령어를 실행시킨다.
```
.\elasticsearch-plugin install analysis-nori
```
plugin 설치 후 ElasticSearch를 재기동 한다.<br>
아래 Postman 등 프로그램을 사용해서 테스트를 진행해볼 수 있다.
<img width="861" alt="스크린샷 2024-05-14 231858" src="https://github.com/yuna1313/yuna1313.github.io/assets/93983333/167d528b-a87d-466b-864d-26e5b6a153b5">

---

참조<br>
[https://velog.io/@colacan100/%EC%97%90%EB%9F%AC%ED%95%B4%EA%B2%B0-Received-plaintext-http-traffic-on-an-https-channel-closing-connection](https://velog.io/@colacan100/%EC%97%90%EB%9F%AC%ED%95%B4%EA%B2%B0-Received-plaintext-http-traffic-on-an-https-channel-closing-connection)<br>
[https://lts0606.tistory.com/496](https://lts0606.tistory.com/496)<br>