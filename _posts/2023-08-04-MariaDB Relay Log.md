---
layout: post
title: MariaDB 운영 복제 방식
date: 2023-08-04 00:00 +0800
last_modified_at: 2023-08-04 00:08:25 +0800
tags: [MariaDB 운영환경 로그 ]
categoie: [MariaDB 운영환경 로그]
topic:  true
---
**MariaDB 운영 log**!
{: .message }

## Bynary Log 간단하게 알아보기.

바이너리 로그는 데이터베이스에서 수행되는 변경 작업들을 기록하는 데 사용되는 파일이다. 주로 데이터베이스 복제, 복구, 보안 및 감사 목적으로 사용하며, 로그 항목은 데이터베이스에서 수행되는 특정 작업에 대한 정보를 포함할 수 있다.
이러한 작업은 예를 들면 INSERT, UPDATE, DELETE 등의 데이터 변경작업일수 있으며 변경 작업, 해당 작업이 수행된 데이터베이스와 테이블 등의 정보가 포함될수 있다.

MariaDB는 다양한 바이너리 로그 형식을 지원하며, 이러한 형식들은 데이터의 변경 내용을 효율적이고 안전하게 기록하고 복구하는 데 도움을 준다. 이형식들은 주로 로그 기록의 크기, 성능, 데이터 무결성 등과 관련이 있다.

* MariaDB는 바이너리 로그라는 파일에 DB 에서 발생한 변경내역을 데이터 폴더에 기본옵션으로 저장한다.
* Bynary Log의 역할및 활용에는 여러가지가 존재하지만 가장 중요한 부분은 바로 복제/복구의 활용이다.

### statement-based 방식의 복제 방식

statement-based 로깅이 활성화되어 있는 경우, 명령문은 실행된 대로 바이너리 로그에 기록된다. Master 서버에서 생성된 임시 테이블은 복제(replica) 서버에도 생성된다. 이 모드는 바이너리 로그 크기를 최소화하려는 경우나 Master서버와 Slave서버가 동일한 데이터를 가지며 모든 사용되는 함수가 결정론적(deterministic)인 경우에만 권장된다. (동일한 인자로 반복 가능한 함수). 타임스탬프나 auto_increment를 사용하는 문장과 테이블은 statement-based 로깅과 함께 사용하는것을 권장한다.

### Row 방식의 복제 방식
DML(데이터 조작 언어) 문은 이진 로그에 기록되지 않는다. 대신 각 행별로 수행되는 각 insert, update 또는 delete 작업이 개별적으로 이진 로그에 기록된다. DDL(데이터 정의 언어) 문은 여전히 이진 로그에 기록된다.

행 기반 로깅은 다른 로그 형식보다 더 많은 저장 공간을 사용하지만, 가장 안전한 방법이다. 

원본 쿼리를 로그로 볼 수 있도록 하려면, 주로 mariadb-binlog과 함께 사용되는 주석이 달린 행 이벤트를 활성화할 수 있다. --binlog-annotate-row-events 옵션을 사용하여 이 이벤트를 표시할 수 있다. 이 옵션은 default로 활성화가 되어있다.

### Mixed 방식의 복제방식
statement-based 로깅과 row-based 로깅을 조합하여 사용한다. 기본적으로는 statement-based 로깅이 사용되지만, 서버가 특정 명령문이 statement-based 로깅에 적합하지 않을 수 있다고 MariaDB 서버가 판단하면 대신 row-based 로깅을 사용한다. 안전하지 않은 명령문의 목록은 Statement-based Replication을 위한 Unsafe Statements: Unsafe Statements에서 확인할 수 있다.

한 트랜잭션 내에서 일부 명령문은 row 로깅으로 기록되고 다른 명령문은 statement-based 로깅으로 기록될 수 있는 방식이며,
my.cnf에서 별도로 지정하지 않으면 default 방식으로 Mixed방식의 복제 방식을 사용한다.

```
로그 포맷형식 확인 방법
MariaDB [(none)]> show variables like "%binlog_format%"
    -> ;
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| binlog_format              | MIXED |
| wsrep_forced_binlog_format | NONE  |
+----------------------------+-------+

```

```
Binary logs 확인 방법

MariaDB [(none)]> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       375 |
| mysql-bin.000002 |   4851033 |
| mysql-bin.000003 |       389 |
| mysql-bin.000004 |       385 |
| mysql-bin.000005 |       342 |
+------------------+-----------+

MariaDB 삭제 주기 확인 방법
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| expire_logs_days | 0     |
+------------------+-------+

 0옵션은 "bin log 를 삭제하지 않는다" 이다.

```

다음과 같이 운영 환경에 적용하기로 했다.

```
my.cnf

expire_logs_days=15
```

운영환경에서 disk관련 문제가 일어날수 있으므로 많이 MariaDB를 많이 사용하는 환경은 아니지만, 그래도 적용하여, disk관련 문제를 제한하여 처리 하는 방안으로 두었다.



