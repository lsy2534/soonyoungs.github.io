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

### advertised.listeners


### auto.leader.rebalance.enable

> auto.leader.rebalance.enable 설정이 활성화되면 백그라운드 스레드가 일정 간격(leader.imbalance.check.interval.seconds로 설정 가능)으로 파티션 리더의 interval를 확인합니다. 만약 브로커의 리더 불균형 비율이 leader.imbalance.per.broker.percentage를 초과하면, 해당 파티션의 선호 리더로 rebalance 트리거가 발생하게 된다.

> 만약 비활성화로 변경하면, 리더 균형 조정이 자동으로 수행되지 않으므로, 브로커에 걸린 리더의 부하가 균형적으로 분산되지 않을 수 있습니다. 이 경우에는 수동으로 리더 균형을 조정해야 하며, 이를 놓치거나 늦게 조치하게 되면 클러스터 전체에 대한 부하 분산 및 안정성 문제가 발생할 수 있습니다. 예를 들어, 일부 브로커에서 리더의 부하가 매우 높은 경우 해당 브로커가 장애를 일으킬 가능성이 높아집니다. 또한 다른 브로커는 리더 부하가 낮아져 있지만 리더 균형 조정이 수동으로 수행되지 않는 경우 해당 브로커의 자원이 낭비될 수 있습니다. 따라서 auto.leader.rebalance.enable 설정을 활성화하여 리더 균형 조정을 자동으로 수행하는 것이 좋습니다.


### control.plane.listener.name 

컨트롤러와 브로커 간 통신에 사용되는 리스너의 이름입니다. 브로커는 control.plane.listener.name을 이용하여 컨트롤러로부터의 연결 요청을 받기 위해 리스너 목록에서 엔드포인트를 찾습니다. 예를 들어, 브로커의 구성이 다음과 같다면:

listeners = INTERNAL://192.1.1.8:9092, EXTERNAL://10.1.1.5:9093, CONTROLLER://192.1.1.8:9094
listener.security.protocol.map = INTERNAL:PLAINTEXT, EXTERNAL:SSL, CONTROLLER:SSL
control.plane.listener.name = CONTROLLER

브로커는 시작시 "192.1.1.8:9094"에서 "SSL" 보안 프로토콜을 사용하여 듣기를 시작합니다. 컨트롤러 측에서는 zookeeper를 통해 브로커의 발행된 엔드포인트를 발견할 때, control.plane.listener.name을 사용하여 브로커에 연결하기 위한 엔드포인트를 찾습니다. 예를 들어, 브로커의 발행된 엔드포인트가 다음과 같고:

"endpoints": ["INTERNAL://broker1.example.com:9092","EXTERNAL://broker1.example.com:9093","CONTROLLER://broker1.example.com:9094"]

컨트롤러의 구성이 다음과 같다면:

listener.security.protocol.map = INTERNAL:PLAINTEXT, EXTERNAL:SSL, CONTROLLER:SSL
control.plane.listener.name = CONTROLLER

컨트롤러는 "broker1.example.com:9094"에서 "SSL" 보안 프로토콜을 사용하여 브로커에 연결합니다. 명시적으로 구성되지 않으면 기본값은 null이며 컨트롤러 연결에 대한 전용 엔드포인트가 없습니다. 명시적으로 구성하는 경우 inter.broker.listener.name 값과 동일할 수 없습니다.