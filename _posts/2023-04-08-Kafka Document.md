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

delete.topic.enable=true

#early.start.listeners=CONTROLLER,INTERNAL
#leader.imbalance.check.interval.seconds=300
#leader.imbalance.per.broker.percentage=10
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


### early.start.listeners

early.start.listeners는 authorizer가 초기화 작업을 완료하기 전에 시작될 수 있는 리스너 이름 목록을 쉼표로 구분하여 지정하는 속성입니다. 이 속성은 StandardAuthorizer와 같이 ACL을 메타데이터 로그에 저장하는 authorizer가 클러스터 자체에 부트스트랩에 의존하는 경우에 유용합니다. 기본적으로는 controller.listener.names에 포함된 모든 리스너도 early start listeners로 구성됩니다. 외부 트래픽을 수신하는 리스너는 이 목록에 나타나면 안됩니다.

ACL은 Access Control List의 약자로, 접근 제어 목록을 의미합니다. 즉, 사용자나 프로세스가 어떤 자원에 접근할 때, 해당 자원에 대한 접근 권한이 있는지를 확인하는 보안 메커니즘입니다. 카프카에서 ACL을 사용하면, 특정 topic에 대한 읽기/쓰기 권한을 가진 사용자나 애플리케이션만 해당 주제에 접근할 수 있도록 제어할 수 있습니다. 이를 통해 카프카의 보안을 강화할 수 있습니다.

early.start.listeners=CONTROLLER

### leader.imbalance.check.interval.seconds 

파티션 리더 불균형을 확인하는 빈도를 결정합니다. 예를 들어, 이 속성이 300으로 설정되어 있으면, Kafka는 5분마다 파티션 리더 불균형을 확인합니다. 이 속성은 브로커 리더 불균형 문제를 신속하게 탐지하고 해결하기 위해 사용됩니다.

### leader.imbalance.per.broker.percentage

하나의 브로커에 대한 파티션 리더 비율이 다른 브로커들에 비해 얼마나 많이 불균형한지를 결정합니다. 예를 들어, 만약 이 속성이 10으로 설정되어 있고 브로커 A, B, C가 있고, 각각의 리더 파티션 수가 100, 150, 50이라면, B 브로커가 가장 많은 파티션 리더를 가지고 있기 때문에 불균형 정도는 (150-100)/100 * 100% = 50%가 됩니다. 이 속성은 브로커 리더 불균형 문제를 해결하기 위해 사용됩니다.

### listeners

리스닝할 URI 목록 및 리스너 이름을 쉼표로 구분하여 나열합니다. 만약 리스너 이름이 보안 프로토콜이 아니라면, listener.security.protocol.map도 설정해야 합니다. 리스너 이름과 포트 번호는 고유해야 합니다. 모든 인터페이스에 바인드하려면 호스트 이름으로 0.0.0.0을 지정하십시오. 기본 인터페이스에 바인드하려면 호스트 이름을 비워두십시오. 올바른 리스너 목록의 예시:
PLAINTEXT://myhost:9092,SSL://:9091
CLIENT://0.0.0.0:9092,REPLICATION://localhost:9093