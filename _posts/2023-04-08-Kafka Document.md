---
layout: post
title: Apache Kafka broker config
date: 2023-04-08 00:00 +0800
last_modified_at: 2023-04-08 00:08:25 +0800
tags: [Kafka, kafka , Kafka.cnf, Apache Kakfa Document broker]
categoie: [Kafka]
toc:  true
---
**Apache Kafka broker config**!
{: .message }

# kafka Document 

### 실제 적용
```
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=1

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092

# Hostname and port the broker will advertise to producers and consumers. If not set,
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
advertised.listeners=PLAINTEXT://172.23.6.4:9092
advertised.host.name=broker-1.example.com
advertised.port=9094
compression.type=snappy

controller.quorum.election.backoff.max.ms=1000
# 
controller.quorum.election.timeout.ms=1000

controller.quorum.election.backoff.max.ms=2000

delete.topic.enable=false

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600

```

### broker.id

Kafka 클러스터 내에서 유일한 브로커를 식별하는 데 사용되는 설정입니다. 각 브로커는 서로 다른 ID를 가져야하며 일반적으로 0부터 시작하여 증가하는 숫자로 구성됩니다. 이 값은 server.properties 파일에서 설정할 수 있습니다. 클러스터 내의 모든 브로커는 같은 zookeeper.connect 설정을 공유하고 있어야합니다. 이렇게 함으로써 각 브로커는 클러스터 내의 다른 브로커를 찾고 클러스터에서 새로운 브로커가 추가될 때 다른 브로커에게 알릴 수 있습니다.
broker.id=-1은 기본적으로 자동 설정되며, ZooKeeper를 사용하여 broker.id를 동적으로 할당하고 싶을 때 유용합니다.

### auto.leader.rebalance.enable

auto.leader.rebalance.enable 설정이 활성화되면 백그라운드 스레드가 일정 간격(leader.imbalance.check.interval.seconds로 설정 가능)으로 파티션 리더의 interval를 확인합니다. 만약 브로커의 리더 불균형 비율이 leader.imbalance.per.broker.percentage를 초과하면, 해당 파티션의 선호 리더로 rebalance 트리거가 발생하게 된다.

만약 비활성화로 변경하면, 리더 균형 조정이 자동으로 수행되지 않으므로, 브로커에 걸린 리더의 부하가 균형적으로 분산되지 않을 수 있습니다. 이 경우에는 수동으로 리더 균형을 조정해야 하며, 이를 놓치거나 늦게 조치하게 되면 클러스터 전체에 대한 부하 분산 및 안정성 문제가 발생할 수 있습니다. 예를 들어, 일부 브로커에서 리더의 부하가 매우 높은 경우 해당 브로커가 장애를 일으킬 가능성이 높아집니다. 또한 다른 브로커는 리더 부하가 낮아져 있지만 리더 균형 조정이 수동으로 수행되지 않는 경우 해당 브로커의 자원이 낭비될 수 있습니다. 따라서 auto.leader.rebalance.enable 설정을 활성화하여 리더 균형 조정을 자동으로 수행하는 것이 좋습니다.


### control.plane.listener.name 

컨트롤러(리더브로커)와 브로커 간 통신에 사용되는 리스너의 이름입니다. 브로커는 control.plane.listener.name을 이용하여 컨트롤러로부터의 연결 요청을 받기 위해 리스너 목록에서 엔드포인트를 찾습니다. 예를 들어, 브로커의 구성이 다음과 같다면:

listeners = INTERNAL://192.1.1.8:9092, EXTERNAL://10.1.1.5:9093, CONTROLLER://192.1.1.8:9094
listener.security.protocol.map = INTERNAL:PLAINTEXT, EXTERNAL:SSL, CONTROLLER:SSL
control.plane.listener.name = CONTROLLER

브로커는 시작시 "192.1.1.8:9094"에서 "SSL" 보안 프로토콜을 사용하여 듣기를 시작합니다. 컨트롤러 측에서는 zookeeper를 통해 브로커의 발행된 엔드포인트를 발견할 때, control.plane.listener.name을 사용하여 브로커에 연결하기 위한 엔드포인트를 찾습니다. 예를 들어, 브로커의 발행된 엔드포인트가 다음과 같고:

"endpoints": ["INTERNAL://broker1.example.com:9092","EXTERNAL://broker1.example.com:9093","CONTROLLER://broker1.example.com:9094"]

컨트롤러의 구성이 다음과 같다면:

listener.security.protocol.map = INTERNAL:PLAINTEXT, EXTERNAL:SSL, CONTROLLER:SSL
control.plane.listener.name = CONTROLLER

컨트롤러는 "broker1.example.com:9094"에서 "SSL" 보안 프로토콜을 사용하여 브로커에 연결합니다. 명시적으로 구성되지 않으면 기본값은 null이며 컨트롤러 연결에 대한 전용 엔드포인트가 없습니다. 명시적으로 구성하는 경우 inter.broker.listener.name 값과 동일할 수 없습니다.

### controller.quorum.election.backoff.max.ms

카프카 클러스터에서 컨트롤러의 선출 과정에서 사용되는 백오프(backoff) 시간의 최대값을 나타내는 설정입니다.

카프카 클러스터에서 컨트롤러는 여러 브로커 중에서 선출되어 클러스터 전체의 메타데이터 관리와 리더 선출 등의 역할을 수행합니다. 만약 현재 선출된 컨트롤러가 고장나거나 클러스터에 새로운 브로커가 추가된 경우, 새로운 컨트롤러를 선출하기 위한 선거 과정이 시작됩니다.

이때, 백오프 시간은 컨트롤러 선출 후보 브로커들이 선거를 요청한 후에 다시 선거를 요청하기 전까지 대기하는 시간을 의미합니다. 백오프 시간이 짧을수록 빠른 선출이 가능하지만, 여러 브로커들이 동시에 선거를 요청하는 경우에는 서로간에 충돌이 발생할 수 있습니다. 반면에 백오프 시간이 길면 충돌은 방지할 수 있지만, 선출 과정이 길어져 클러스터의 가용성이 떨어질 수 있습니다.

따라서 controller.quorum.election.backoff.max.ms 설정은 컨트롤러 선거 과정에서 발생하는 충돌을 방지하기 위해, 컨트롤러 후보들이 다음 선거 요청을 보내기까지 기다리는 최대 대기 시간을 설정하는 값입니다. 이 값이 크면 충돌은 방지할 수 있지만, 선거 과정의 속도가 느려지게 됩니다.

### controller.quorum.election.timeout.ms

리더로부터 가져올 수 없는 경우 새로운 선거를 트리거하기 전에 기다릴 수 있는 최대 시간(밀리초)입니다.
'controller.quorum.election.backoff.max.ms' 같이 사용하여 충돌이 발생하는 것을 방지할 수 있다. 

### controller.quorum.election.backoff.max.ms
다수의 카프카 컨트롤러가 서로 경쟁하는 경우에 사용됩니다. 이 속성은 현재 리더에서 메시지를 가져올 수 없는 경우 새로운 선거를 시작하기 위해 기다릴 수 있는 최대 시간을 지정합니다.

또한 이 속성은 다수의 카프카 컨트롤러가 현재 레플리카 세트를 제대로 동기화하지 못했을 때 새로운 리더 Epoch가 있을 수 있는지 확인하기 위해 이전 Epoch와 다른 Epoch로 쿼리를 수행하는 데 사용됩니다.

Epoch란 Broker나 Controller에서 사용되는 일련 번호(sequence number)를 의미합니다. Epoch는 Kafka의 레코드(log record)와 같은 데이터의 순서와 동기화를 유지하는 데 사용됩니다. Broker 또는 Controller는 이전 Epoch의 값을 알고 있다면 이전에 처리되지 않은 요청에 대한 처리를 건너뛰고 바로 새로운 Epoch로 넘어갈 수 있습니다

### advertised.port

Kafka에서 advertised.port는 브로커가 자신의 IP 주소와 포트 번호를 클라이언트에게 알리는 데 사용되는 설정 속성입니다. 이 속성은 클라이언트가 브로커에 연결하기 위해 사용할 수 있는 IP 주소와 포트 번호를 지정합니다. 예를 들어, 브로커가 내부 IP 주소와 9092 포트로 바인딩되어 있고, 외부에서는 공인 IP 주소와 9094 포트로 접근할 수 있다면, advertised.port 설정은 9094로 지정됩니다. 이렇게 함으로써 외부 클라이언트는 공인 IP 주소와 9094 포트로 브로커에 연결할 수 있습니다. 또한, advertised.port 설정은 클라우드 환경에서 브로커가 가상 IP 주소와 포트로 연결되어 있는 경우에도 유용합니다. 이 경우 브로커가 실제로 바인딩되어 있는 IP 주소와 포트가 아닌, 가상 IP 주소와 포트로 클라이언트에게 연결 정보를 전달할 수 있습니다.

### compression.type

카프카에서 메시지 압축에 사용할 압축 타입을 지정하는 설정입니다. 기본값은 "producer"이며, 메시지를 생산자에서 압축하여 카프카에 전송합니다.

compression.type으로 설정할 수 있는 값으로는 "none", "gzip", "snappy", "lz4", "zstd" 등이 있으며, 각각의 값은 다른 압축 알고리즘을 나타냅니다.

압축을 사용하면 네트워크 대역폭을 줄일 수 있으며, 메시지를 저장하는 디스크 공간을 절약할 수 있습니다. 그러나 압축을 사용하면 CPU 자원이 추가로 필요하므로, 성능에 영향을 줄 수 있습니다. 따라서 압축을 사용할지 여부는 사용 사례에 따라 적절히 고려해야 합니다.


### delete.topic.enable

기본값은 false이며, 이 경우에는 Kafka 서버에서 토픽 삭제 요청이 거부됩니다. 이것은 실수로 데이터를 삭제하는 것을 방지하고, 데이터 손실을 예방하기 위해 Kafka에서 권장하는 값입니다.
![image](https://user-images.githubusercontent.com/57931679/230720262-2bc3e3e8-e67e-490d-8fc1-75a7a725dfdb.png)

하지만, delete.topic.enable 값을 true로 설정하면 Kafka에서 토픽을 삭제할 수 있습니다. 이 경우에는 삭제된 토픽에 저장된 데이터는 완전히 삭제되며, 데이터 복구가 불가능합니다. 따라서, 신중히 사용해야 하며, 특히 운영 환경에서는 조심해서 사용해야 합니다.

