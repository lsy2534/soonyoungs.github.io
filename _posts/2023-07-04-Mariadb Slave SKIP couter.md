---
layout: post
title: MariaDB Slave SKIP couter 
date: 2023-07-04 00:00 +0800
last_modified_at: 2023-07-04 00:08:25 +0800
tags: [MariaDB Slave ]
categoie: [MariaDB]
topic:  true
---
**MariaDB Slave**!
{: .message }


### MariaDB Slave SKIP Couter

에러 로그를 찍기는 어려운 환경이라 에러를 가져오지 못한건 아쉬운일이다. 으아.. 캡처본이라도 뽑아주지..(약간 치사하다는 생각이 좀 든다.)

일단 저번 포스트 중 Maria DB 이중화를 한적이있다. 

https://lsy2534.github.io/soonyoungs.github.io/tags/#mariadb-%EC%9D%B4%EC%A4%91%ED%99%94

Sami Sync도 중 에러가 발생하였다. sami sync(Master에서 DB 를 전체 데이터 베이스를 dump 받아, Slave 서버에 import 하는 행위) 

System DB(mysql, information_db) 이러한 Mysql 자체에 시스템 테이블이 동기화가 되지않아 Slave start시 문제가 발생했다.

에러를 해결하는 방법은 

이벤트가 트랜잭션인 경우 전체 트랜잭션을 건너 뛰도록 하는 방법이다. . 비트랜잭션 엔진에서 이벤트는 항상 단일 문이고,

이 명령문은 복제 스레드가 실행 중이 아닌 경우에만 유효하다. 그렇지 않으면 오류가 발생하는경우가 생긴다.

```

SHOW SLAVE STATUS \G
...
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE;

```
