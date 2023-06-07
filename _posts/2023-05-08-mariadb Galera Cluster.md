---
layout: post
title: MariaDB Galera Cluster
date: 2023-05-24 00:00 +0800
last_modified_at: 2023-05-24 00:08:25 +0800
tags: [ MariaDB Galera Cluster, CentOS 8 Stream, MariaDB]
categoie: [Server Date]
toc:  true
---
**MariaDB Galera Cluster**!
{: .message }


# MariaDB Galera Cluster  

MariaDB Galera Cluster는 MariaDB를 위한 가상 동기 다중 기본 클러스터이며, Linux에서만 사용할 수 있다. InnoDB 스토리지 엔진 만 지원한다.( MyISAM 및 MariaDB 10.6 부터 Aria 에 대한 실험적 지원이 있지만 ![wsrep_replicate_myisam](https://mariadb.com/kb/en/galera-cluster-system-variables/#wsrep_replicate_myisam) 시스템 변수 또는 MariaDB 10.6 부터 wsrep_mode 시스템 변수 참조 ).
* InnoDB 스토리엔진 : Mysql 데이터베이스 관리 시스템에서 사용되는 스토리엔진 중 하나 입니다. 스토리지 엔진은 데이터 베이스 시스템에서 데이터를 저장, 관리, 검색하는데 사용되는 핵심 컴포넌트이다.

InnoDB 스토리 엔진은 다음과 같은 주요 특징을 가지고있다.

1. 트랜잭션 지원 : InnoDB는 ACID(원자성, 일관성, 격리성, 지속성)특성을 준수하여 데이터베이스 트랜잭션을 안전하게 관리합니다. 이는 데이터의 일관성과 신뢰성을 보장하며, 데이터의 동시 접근을 관리하기 위해 격리 수준을 제공한다.
2. 행 레벨 잠금 : InnoDB는 행 레벨 잠금(row-level locking)을 지원하여 동시성을 높일 수 있습니다. 이는 여러 사용자가 동시에 데이터를 수정하거나 검색할 수 있도록 한다.
3. 외래키 지원 : InnoDB는 외래키(Foreign Key) 제약 조건을 지원하여 데이터의 무결성을 보장한다. 이를 통해서 데이터 간의 관계를 정의하고 유지할수 있다.
4. 버퍼 풀 관리 : InnoDB는 데이터베이스의 성능을 향상시키기 위해 메모리 버퍼 풀을 사용한다. 이를 통해서 데이터 베이스 작업을 디스크 I/O보다 빠르게 수행할 수 있다. 
5. 복구 기능 : InnoDB는 시스템 장애 또는 비정상적인 종료 시 데이터베이스의 일관성을 복구할 수 있는 기능을 제공한다. 이를 통해 데이터 손실을 방지하고 데이터의 안전성을 보장할 수 있다.

### Galera 특징

* 가상 동기식 복제
* Active-Active 다중 기본 토플로지
* 모든 클러스터 노드에 대한 읽기 및 쓰기
* 클러스터 제어, 실패한 노드가 클러스터에서 삭제(스케일 IN)
* 행 에 대한 병렬 복제 및 처리
* 직접 클라이언트 연결



