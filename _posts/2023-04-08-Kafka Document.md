---
layout: post
title: Apache Zoopkeeper config
date: 2023-04-08 00:00 +0800
last_modified_at: 2023-04-08 00:08:25 +0800
tags: [Kafka, kafka , Kafka.cnf, Apache Kakfa Document]
categoie: [Kafka]
toc:  true
---
**Apache Kafka config**!
{: .message }

kafka Document 

### advertised.listeners(필수)


### auto.leader.rebalance.enable(필수)

> 자동 리더 균형 기능을 활성화하는 설정입니다. 이 설정이 활성화되면 백그라운드 스레드가 일정 간격(leader.imbalance.check.interval.seconds로 설정 가능)으로 파티션 리더의 분포를 확인합니다. 만약 브로커의 리더 불균형 비율이 leader.imbalance.per.broker.percentage를 초과하면, 해당 파티션의 선호 리더로 리더 균형 조정이 트리거됩니다