---
layout: post
title: PostgreSQL Queue 기반 비동기 파일 처리 구조
date: 2026-11-12 00:00 +0800
last_modified_at: 2026-11-12 10:00:00 +0800
tags: [PostgreSQL, Queue, Database, Backend, Async]
categoie: [Database, PostgreSQL]
topic: true
---

**PostgreSQL Queue 기반 비동기 처리 구조**!
{: .message }

## 개요

대량의 파일이나 데이터를 API 요청 시점에 모두 처리하면 응답 지연, 시스템 부하, 타임아웃 등의 문제가 발생할 수 있다.

이러한 문제를 해결하기 위해 PostgreSQL을 Queue 저장소로 활용하고, 실제 데이터 처리는 별도의 Worker가 수행하는 비동기 처리 구조를 학습했다.

PostgreSQL Queue는 별도의 메시지 브로커 없이 PostgreSQL 테이블을 Queue처럼 활용하는 방식이다.

핵심은 **API는 처리 요청을 Queue에 등록하고, Worker는 Queue에서 작업을 가져와 실제 처리를 수행하는 것**이다.

## 1. PostgreSQL Queue란?

PostgreSQL Queue는 일반적인 테이블을 작업 대기열(Queue)로 활용하는 구조이다.

Kafka와 같은 메시지 브로커를 별도로 구축하지 않고도 데이터베이스의 트랜잭션과 잠금 기능을 활용하여 작업을 관리할 수 있다.

### 주요 특징

| 구분 | 설명 |
|---|---|
| Queue 저장소 | PostgreSQL 테이블 |
| 데이터 등록 | INSERT |
| 작업 조회 | SELECT |
| 작업 상태 관리 | UPDATE |
| 동시성 제어 | Row Lock |
| 트랜잭션 | PostgreSQL Transaction |
| 재시도 | 실패 상태 및 Retry Count 관리 |
| 처리 방식 | Scheduler + Worker |

### PostgreSQL Queue를 사용하는 이유

- 별도의 메시지 브로커 구축 없이 처리 가능
- 기존 PostgreSQL 인프라 활용 가능
- Queue와 업무 데이터 간 트랜잭션 관리가 용이
- 작업 상태와 실패 이력을 DB에서 관리 가능
- 소규모 비동기 처리 시스템에 적합

---

## 2. 전체 시스템 아키텍처

이번 학습에서 구현하고자 하는 구조는 파일 등록 API를 통해 작업을 Queue에 등록하고, 별도의 Worker가 Queue를 처리하여 최종 DB에 적재하는 방식이다.

```text
                  Client
                    |
                    v
             File Register API
                    |
                    v
          PostgreSQL Queue Table
                    |
                    v
            Scheduler / Worker
                    |
                    v
           Queue 작업 조회
                    |
                    v
          파일 데이터 처리
                    |
                    v
           Target Database
```

### 각 구성 요소의 역할

| 구성 요소 | 역할 |
|---|---|
| File Register API | 파일 등록 요청을 수신 |
| PostgreSQL Queue | 처리할 작업을 저장 |
| Scheduler | Worker 실행 주기 관리 |
| Queue Worker | Queue 작업 조회 및 처리 |
| Target Database | 실제 업무 데이터 적재 |

---

## 3. 처리 흐름

### Step 1. 파일 등록 API

사용자가 파일 등록 API를 호출하면 파일 처리에 필요한 정보를 Queue에 등록한다.

```http
POST /api/files
Content-Type: application/json
```

```json
{
  "fileId": "FILE-20261112-001",
  "fileName": "customer.csv",
  "filePath": "/data/customer.csv"
}
```

API에서는 실제 파일의 데이터 적재까지 수행하지 않는다.

파일 정보와 처리 요청을 Queue에 등록하고 즉시 응답한다.

### Step 2. PostgreSQL Queue 적재

파일 등록 API는 Queue 테이블에 작업을 INSERT 한다.

```sql
INSERT INTO file_queue (
    file_id,
    file_name,
    file_path,
    status,
    retry_count,
    created_at
)
VALUES (
    'FILE-20261112-001',
    'customer.csv',
    '/data/customer.csv',
    'WAITING',
    0,
    NOW()
);
```

Queue에 등록된 데이터는 Worker가 처리할 때까지 대기한다.

### Step 3. Scheduler Queue Worker 실행

Worker는 Scheduler에 의해 일정 주기로 실행된다.

예를 들어 1초 또는 5초마다 Queue를 조회하여 처리할 작업을 가져올 수 있다.

```text
Scheduler
    |
    +-- Worker 실행
            |
            +-- WAITING Queue 조회
            |
            +-- 작업 데이터 확보
            |
            +-- 처리 상태 변경
            |
            +-- 파일 데이터 처리
            |
            +-- Target DB 적재
            |
            +-- Queue 상태 변경
```

Worker는 Queue에 등록된 작업을 하나씩 처리하거나 일정 건수만큼 Batch로 처리할 수 있다.

### Step 4. DB 적재

파일 데이터 처리가 완료되면 실제 업무 테이블에 데이터를 적재한다.

```sql
INSERT INTO customer (
    customer_id,
    customer_name
)
VALUES (
    'C001',
    '홍길동'
);
```

적재가 정상적으로 완료되면 Queue 상태를 `SUCCESS`로 변경한다.

```sql
UPDATE file_queue
SET status = 'SUCCESS',
    completed_at = NOW()
WHERE file_id = 'FILE-20261112-001';
```

---

## 4. Queue 테이블 설계

Queue 테이블은 작업 상태와 처리 이력을 관리할 수 있도록 설계해야 한다.

```sql
CREATE TABLE file_queue (
    queue_id BIGSERIAL PRIMARY KEY,
    file_id VARCHAR(100) NOT NULL,
    file_name VARCHAR(255),
    file_path VARCHAR(500),

    status VARCHAR(20) NOT NULL DEFAULT 'WAITING',

    retry_count INTEGER DEFAULT 0,
    error_message TEXT,

    created_at TIMESTAMP DEFAULT NOW(),
    started_at TIMESTAMP,
    completed_at TIMESTAMP
);
```

### Queue 상태 관리

| 상태 | 설명 |
|---|---|
| WAITING | 처리 대기 |
| PROCESSING | Worker 처리 중 |
| SUCCESS | 정상 처리 완료 |
| FAILED | 처리 실패 |

상태를 구분하면 현재 처리 상황과 실패 작업을 쉽게 확인할 수 있다.

---

## 5. Queue Worker 동시성 제어

Worker를 여러 개 실행하는 경우 동일한 Queue 작업을 중복으로 처리하지 않도록 해야 한다.

PostgreSQL에서는 `FOR UPDATE SKIP LOCKED`를 활용하여 이미 다른 Worker가 잠근 작업을 건너뛸 수 있다.

```sql
SELECT queue_id, file_id, file_path
FROM file_queue
WHERE status = 'WAITING'
ORDER BY queue_id
LIMIT 100
FOR UPDATE SKIP LOCKED;
```

### SKIP LOCKED의 역할

- Worker A가 특정 Queue Row를 잠금
- Worker B는 잠긴 Row를 기다리지 않고 건너뜀
- 다른 대기 작업을 가져와 처리
- Worker 간 중복 작업 수행을 방지

단, Queue 작업을 조회한 후 잠금을 해제하기 전에 `PROCESSING` 상태로 변경하고, 상태 변경과 작업 확보를 하나의 트랜잭션으로 처리해야 한다.

---

## 6. 대량 처리: 1,000건 Batch

Queue에 1,000건의 파일이 등록되는 경우 매번 하나씩 처리하는 방식보다 Batch 단위 처리가 효율적일 수 있다.

```text
Queue
 ├─ FILE-001
 ├─ FILE-002
 ├─ FILE-003
 ├─ ...
 └─ FILE-1000
```

Worker는 일정 건수씩 Queue를 가져온다.

```text
Worker
  |
  +-- Batch 1 : 100건
  |
  +-- Batch 2 : 100건
  |
  +-- Batch 3 : 100건
  |
  +-- ...
  |
  +-- Batch 10 : 100건
```

Batch Size를 100으로 설정하면 1,000건의 작업을 10개 Batch로 나누어 처리할 수 있다.

### Batch 처리 시 고려사항

| 항목 | 고려사항 |
|---|---|
| Batch Size | 처리량과 메모리 사용량 균형 |
| Worker 수 | 동시 처리량 조절 |
| DB Connection | Worker별 Connection Pool 관리 |
| Transaction | Batch 단위 커밋 전략 |
| Retry | 실패 작업만 재처리 |
| Idempotency | 중복 적재 방지 |

Batch Size와 Worker 수를 무조건 증가시키는 것이 성능 향상으로 이어지는 것은 아니다.

DB Connection, I/O, CPU, Lock 경합을 함께 고려해야 한다.

---

## 7. PostgreSQL Queue와 Kafka 비교

PostgreSQL Queue와 Kafka는 모두 비동기 처리에 사용할 수 있지만 목적과 특성이 다르다.

| 구분 | PostgreSQL Queue | Kafka |
|---|---|---|
| 저장소 | PostgreSQL Table | Kafka Topic |
| 데이터 저장 | DB Row | 메시지 Log |
| 처리 방식 | Polling 기반 | Consumer 기반 |
| 동시성 | Row Lock | Partition |
| 상태 관리 | DB 상태 컬럼 | Consumer Offset |
| 재시도 | DB 상태 및 Retry | 재시도 Topic 등 |
| 트랜잭션 | DB 트랜잭션 | Kafka 트랜잭션 |
| 운영 복잡도 | 상대적으로 낮음 | 상대적으로 높음 |
| 대규모 이벤트 처리 | DB 부하 고려 | 대규모 처리에 적합 |

### 어떤 상황에서 PostgreSQL Queue를 사용할까?

- 기존 PostgreSQL 환경을 활용하고 싶은 경우
- 작업 요청과 업무 데이터를 함께 관리하는 경우
- 수백~수천 건 수준의 작업을 처리하는 경우
- 복잡한 메시지 브로커 운영을 피하고 싶은 경우
- Queue 작업 상태를 SQL로 쉽게 조회하고 싶은 경우

Kafka는 대규모 이벤트 스트리밍, 여러 Consumer 그룹, 이벤트 재처리 및 높은 메시지 처리량이 필요한 환경에서 더 적합할 수 있다.

---

## 8. 성능 측면에서 고려해야 할 사항

PostgreSQL Queue는 구현이 간단하지만 Queue 자체가 DB 테이블이기 때문에 데이터가 누적되면 성능 문제가 발생할 수 있다.

### 8.1 Queue Index

Queue 조회 조건에 맞는 Index를 설계해야 한다.

```sql
CREATE INDEX idx_file_queue_waiting
ON file_queue (queue_id)
WHERE status = 'WAITING';
```

Partial Index를 사용하면 WAITING 상태의 작업만 Index 대상으로 관리할 수 있다.

### 8.2 Queue 데이터 정리

처리 완료된 Queue 데이터를 무제한 보관하면 테이블과 Index가 커질 수 있다.

- SUCCESS 데이터 일정 기간 보관
- 오래된 Queue 데이터 Archive
- 불필요한 데이터 삭제
- VACUUM 및 Autovacuum 상태 확인

### 8.3 Polling 주기

Worker가 너무 짧은 주기로 Queue를 조회하면 불필요한 SQL 호출이 증가한다.

반대로 Polling 주기가 너무 길면 작업 대기 시간이 증가한다.

처리량과 응답 지연을 고려하여 Scheduler 주기를 조정해야 한다.

### 8.4 Transaction 범위

Queue 작업을 확보하는 Transaction과 실제 파일 처리 Transaction을 구분하는 것이 중요하다.

파일 처리 전체 시간 동안 Queue Row Lock을 유지하면 Lock 경합이 증가할 수 있다.

일반적으로 다음과 같은 방식으로 설계할 수 있다.

1. Queue 작업 확보
2. PROCESSING 상태 변경 및 Commit
3. 실제 파일 처리
4. 업무 데이터 적재
5. Queue SUCCESS 처리

단, 업무 데이터 적재와 Queue 완료 상태가 서로 다른 Transaction이면 중간 장애 시 중복 처리가 발생할 수 있으므로 Idempotency를 고려해야 한다.

---

## 9. 장애 및 재처리 설계

Worker가 처리 중 종료되거나 DB 오류가 발생하는 경우 Queue 작업이 영구적으로 PROCESSING 상태에 남을 수 있다.

이를 방지하기 위해 다음 항목을 고려한다.

- Worker 실행 시간 기록
- Processing Timeout 관리
- 일정 시간 이상 PROCESSING 상태인 작업 재확인
- Retry Count 증가
- 재시도 최대 횟수 설정
- 실패 원인 기록
- 최종 실패 작업 별도 관리

특히 파일 데이터 적재 시 중복 INSERT가 발생하지 않도록 Unique Key, UPSERT 또는 별도 처리 이력 테이블을 활용해야 한다.

---

## 10. 오늘 학습한 내용 정리

오늘은 PostgreSQL을 Queue 저장소로 활용하여 비동기 파일 처리 구조를 설계하는 방법을 학습했다.

핵심 흐름은 다음과 같다.

```text
File Register API
        |
        v
PostgreSQL Queue
        |
        v
Scheduler Queue Worker
        |
        v
Batch Processing
        |
        v
Target Database
```

### 핵심 학습 내용

- PostgreSQL Table을 Queue로 활용할 수 있다.
- API와 실제 데이터 적재를 분리하여 비동기 처리할 수 있다.
- Scheduler를 통해 Worker 실행 주기를 관리할 수 있다.
- `FOR UPDATE SKIP LOCKED`로 Worker 간 작업 중복을 방지할 수 있다.
- Batch Size와 Worker 수를 통해 처리량을 조절할 수 있다.
- Queue 상태와 Retry Count를 통해 작업을 관리할 수 있다.
- Kafka와 비교하여 시스템 규모와 운영 복잡도에 맞는 Queue를 선택할 수 있다.

### 다음 실습 목표

실제 Spring Boot 기반으로 PostgreSQL Queue를 구현한다.

1. 파일 등록 API 구현
2. PostgreSQL Queue 테이블 생성
3. Queue 데이터 INSERT
4. Scheduler Queue Worker 구현
5. Queue에서 1,000건 Batch 처리
6. Worker에서 Target DB 적재
7. Queue 상태 및 실패 작업 관리

이번 실습에서는 **파일 등록 API부터 PostgreSQL Queue를 거쳐 최종 DB에 적재되는 과정까지** 구현하는 것을 목표로 한다.
