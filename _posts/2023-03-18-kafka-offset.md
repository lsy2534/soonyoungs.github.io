---
layout: post
title: Kafka Options
date: 2023-03-20 00:00 +0800
last_modified_at: 2023-03-20 00:08:25 +0800
tags: [kafka, kafka-option]
categoies: [kafka]
toc:  true
---
**Kafka Option**!
{: .message }

Apache Kafka는 다양한 옵션을 제공하여 더욱 유연하고 세밀한 설정이 가능하다. 일부 주요 옵션에 대해 알아본다.
카프카(Kafka)는 분산 스트리밍 플랫폼으로, 대용량의 실시간 데이터 스트림 처리에 사용된다. 카프카는 다양한 옵션을 제공하여 사용자가 필요에 맞게 설정할 수 있도록 해준다. 아래와 같이 카프카 옵션을 자세히 알아보고자 한다.

## Kafka Option
1. Broker 관련 옵션:
broker.id: 카프카 브로커의 고유 식별자를 지정한다.
listeners: 카프카 브로커가 클라이언트 요청을 수신할 IP 및 포트를 지정합니다.
log.dirs: 카프카 브로커가 메시지 로그를 저장할 디렉토리를 지정합니다.

2. Topic 관련 옵션:
replication.factor: 각 파티션에 대해 복제본 수를 지정한다.
partition.assignment.strategy: 브로커에 새로운 토픽이 생성될 때 파티션 할당 방식을 지정합니다.
retention.ms: 메시지를 보관할 시간을 지정합니다.
