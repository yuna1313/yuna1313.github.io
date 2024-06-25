---
published: true
title: "[DB] MariaDB Master - Master 설정"
categories:
  - DB
tags:
  - MariaDB
  - MySQL

toc: true,
toc_sticky: true

date: 2024-06-25
last_modified_at: 2024-06-25
---

고객 사 전산팀에서 서버 및 DB 이중화를 요청했다. ~~(몇 년 동안 안 하다가 왜 내가 입사하니까 갑자기..ㅠ)~~ <br>
서버 이중화는 기존에 L4가 세팅되어 있어서 간단한 Nginx 세팅과 서비스만 올리면 되었고 DB는 따로 세팅해주어야 했다.

## 1.Master - Slave
처음에는 Master - Slave로 설정하면 될 줄 알고 세팅해봤는데 문제점이 있었다.<br> 
Slave 서버는 껐다 켜도 동기화가 되지만 Master 서버를 껐다 키면 동기화가 되지 않았다.<br>
내가 바라던 방식은

1. 어느 DB에 데이터를 넣어도 동기화 되어야 한다.
2. 한 개의 DB를 껐다 켜도 동기화가 되어야 한다.

라는 조건이었다.

## 2.Master - Master
그래서 알게된 Master - Master 설정<br>
서로를 서로의 Master이면서 Slave로 설정하는 것이다. 그래서 내가 위에 작성한 내용의 조건을 충족시킬 수 있었다.<br>
설정 방법은 간단하다.

IP가 아래와 같이 설정되어있다고 가정해보자

```
Server_A (10.10.10.11)
Server_B (10.10.10.12)
```

## 3.Server_A 세팅
### 3-1.my.cnf 세팅
먼저 Server_A 부터 세팅해준다.<br>
MariaDB 설정파일인 /etc/mysql/my.cnf에 아래 내용을 추가해준다.


```
[mysqld]
log-bin
server_id=1 
replicate-do-db=mobile_app 
bind-address=0.0.0.0 
```
- log-bin: 바이너리 로그 활성화 시켜주는 설정이다. 로그를 통해 동기화하기 때문에 활성화 시켜줘야 한다.
- server_id: 서버 아이디 설정
- replicate-do-db: 특정 데이터베이스만 동기화되도록 할 수 있는 설정이다. 만약 여러 개 하고 싶다면 아래로 쭉 나열하여 여러 개 작성할 수 있다.
- bind-address: 특정 IP만 지정 허용할 수 있는 설정이다. 만약 0.0.0.0으로 설정하면 모든 IP를 허용하겠다는 의미이다.

### 3-2.Mysql 재실행
설정 내용 저장 후 Mysql을 재실행 해준다.
```
service mysql restart
or
systemctl restart mysql
```

### 3-3.계정 생성
동기화할 계정을 생성해준다.
```
CREATE USER ‘repli’@‘%’ IDENTIFIED BY ‘qwer1234!’	=> 계정 생성
GRANT REPLICATION SLAVE ON *.* TO ‘repli’@‘%’;	=> SLAVE 권한 부여
GRANT ALL PRIVILEGES ON mobile_app.* TO ‘repli’@‘%’;	=> mobile_app 데이터베이스 관련한 모든 권한 부여
FLUSH PRIVILEGES;			=> 수정 내용 저장
```

## 4.Server_B 세팅
### 4-1.my.cnf 세팅
같은 방식으로 Server_B 세팅해준다.<br>
/etc/mysql/my.cnf에 아래 내용을 추가해준다.


```
[mysqld]
log-bin
server_id=2
replicate-do-db=mobile_app 
bind-address=0.0.0.0 
```
### 4-2.Mysql 재실행
설정 내용 저장 후 Mysql을 재실행 해준다.
```
service mysql restart
or
systemctl restart mysql
```

### 4-3.계정 생성
동기화할 계정을 생성해준다.
```
CREATE USER ‘repli’@‘%’ IDENTIFIED BY ‘qwer1234!’	=> 계정 생성
GRANT REPLICATION SLAVE ON *.* TO ‘repli’@‘%’;	=> SLAVE 권한 부여
GRANT ALL PRIVILEGES ON mobile_app.* TO ‘repli’@‘%’;	=> mobile_app 데이터베이스 관련한 모든 권한 부여
FLUSH PRIVILEGES;			=> 수정 내용 저장
```

### 4-4.Salve 설정
먼저 Server_A에 가서 Master 상태를 확인한다.
```
SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File | Position | Binlog_Do_DB | Binlog_Ignore_DB|
+------------------+----------+--------------+------------------+
| mariadb-bin.000001 | 626 | | |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

그리고 Server_B에서 Slave 설정을 해준다.
```
STOP SLAVE;				=> 작동 중일 경우 STOP
CHANGE MASTER TO 
	MASTER_HOST = ‘10.10.10.11’, 	=> MASTER IP
	MASTER_USER = ‘repli’, 		=> MASTER 유저명
	MASTER_PASSWORD = ‘qwer1234!’, 	=> MASTER 비밀번호
	MASTER_LOG_FILE = ‘mariadb-bin.000001’, 	=> MASTER 로그 파일
	MASTER_LOG_POS = 626;		=> MASTER 로그 파일 위치
START SLAVE;			=> SLAVE 실행
SHOW SLAVE STATUS\G;			=> SLAVE 상태 확인
```
## 5.Server_A 세팅
### 5-1.Salve 설정
마지막으로 Server_A에 Slave 설정을 해주면 된다.<br>
Server_B에 가서 Master 상태를 확인한다.
```
SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File | Position | Binlog_Do_DB | Binlog_Ignore_DB|
+------------------+----------+--------------+------------------+
| mariadb-bin.000002 | 635 | | |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

해당 내용을 참고하여 Slave 설정을 해준다.
```
STOP SLAVE;				=> 작동 중일 경우 STOP
CHANGE MASTER TO 
	MASTER_HOST = ‘10.10.10.12’, 	=> MASTER IP
	MASTER_USER = ‘repli’, 		=> MASTER 유저명
	MASTER_PASSWORD = ‘qwer1234!’, 	=> MASTER 비밀번호
	MASTER_LOG_FILE = ‘mariadb-bin.000002’, 	=> MASTER 로그 파일
	MASTER_LOG_POS = 635;		=> MASTER 로그 파일 위치
START SLAVE;			=> SLAVE 실행
SHOW SLAVE STATUS\G;			=> SLAVE 상태 확인
```

## 6.이중화 테스트
이렇게 하면 Master - Master 설정은 끝이다. 이제 한 번 테스트 해보자.

- Server_A (테스트하느라 1,2 데이터가 안 맞음..ㅠ)
![KakaoTalk_20240624_224350625](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/bb5b1ce7-1b84-4a5c-b823-24828e0ccf3f)

- Server_B
![KakaoTalk_20240624_224350625_01](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/c663b8f8-0296-46aa-9444-4b06c87e914e)

- 그리고 Server_A 테이블에 데이터를 입력해준다.
![KakaoTalk_20240624_224350625_02](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/0f158ae5-8622-4f54-876c-6a12aa8fb16a)

- Server_B 테이블에 데이터 들어간 것 확인
![KakaoTalk_20240624_224350625_03](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/611e643e-bf79-4b4b-9604-636984c3db0d)

- 이번엔 Server_B 테이블에 데이터 입력
![KakaoTalk_20240624_224350625_04](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/388be1bb-e2a0-4c94-badd-b1551b8be975)

- Server_A 테이블에 데이터 들어간 것 확인
![KakaoTalk_20240624_224350625_05](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/9f25340f-ded7-4c44-a199-83154561ff9b)

이중화 작업은 처음해봐서 복잡할까봐 걱정했는데 생각보다 단순하여 다행이었다! 다만 서비스 수정하는 과정이 너무 귀찮..ㅠ
