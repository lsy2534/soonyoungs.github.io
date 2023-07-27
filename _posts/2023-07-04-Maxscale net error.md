---
layout: post
title: Maxscale Log error
date: 2023-07-24 00:00 +0800
last_modified_at: 2023-07-24 00:08:25 +0800
tags: [Maxscale ]
categoie: [Maxscale]
topic:  true
---
**MariaDB Master**!
{: .message }


### MariaDB Master

MariaDB maxscale 구성 후 2달이 지나 로그를 확인 한 결과 새벽에 에러가 발생하고 있었다.   
에러 내용은 다음과 같다.


```
2023-07-24 10:54:52   notice : Read 8 user@host entries from 'server2' for service 'Read-Write-Service'.
2023-07-24 21:39:36   error  : (590) [readwritesplit] (Read-Write-Service); Lost connection to the primary server, closing session. Lost connection to primary server while connection was idle. A transaction is active and cannot be replayed. Connection has been idle for 28800 seconds. Error caused by: #HY000: Lost connection to backend server: network error (server2: 220, Connection reset by peer). Last close reason: <none>. Last error:
2023-07-24 21:42:24   error  : (592) [readwritesplit] (Read-Write-Service); Lost connection to the primary server, closing session. Lost connection to primary server while connection was idle. A transaction is active and cannot be replayed. Connection has been idle for 28800 seconds. Error caused by: #HY000: Lost connection to backend server: network error (server2: 220, Connection reset by peer). Last close reason: <none>. Last error:
2023-07-24 21:44:37   error  : (591) [readwritesplit] (Read-Write-Service); Lost connection to the primary server, closing session. Lost connection to primary server while connection was idle. Connection has been idle for 29101 seconds. Error caused by: #HY000: Lost connection to backend server: network error (server2: 220, Connection reset by peer). Last close reason: <none>. Last error:
2023-07-24 23:09:18   error  : (1180) [readwritesplit] (Read-Write-Service); Lost connection to the primary server, closing session. Lost connection to primary server while connection was idle. A transaction is active and cannot be replayed. Connection has been idle for 28800 seconds. Error caused by: #HY000: Lost connection to backend server: network error (server2: 220, Connection reset by peer). Last close reason: <none>. Last error:
2023-07-24 23:09:53   error  : (953) [readwritesplit] (Read-Write-Service); Lost connection to the primary server, closing session. Lost connection to primary server while connection was idle. Connection has been idle for 28800 seconds. Error caused by: #HY000: Lost connection to backend server: network error (server2: 220, Connection reset by peer). Last close reason: <none>. Last error:
2023-07-27 00:13:11   error  : (9113) [readwritesplit] (Read-Write-Service); Lost connection to the primary server, closing session. Lost connection to primary server while connection was idle. Connection has been idle for 28800 seconds. Error caused by: #HY000: Lost connection to backend server: network error (server2: 220, Connection reset by peer). Last close reason: <none>. Last error:

```

약간 보면 뭔가 네트워크쪽에서 끊기는 문제가 발생하고 있는것처럼 보이나, 자세하게 GPT에게 함 물어보았다.

내용은 다음과 같다.

```
[readwritesplit] (읽기-쓰기-서비스); 주 서버와의 연결이 끊겼습니다. 세션을 종료합니다. 연결이 유휴 상태인 동안 주 서버와의 연결이 끊겼습니다. 연결이 28800초 동안 유휴 상태였습니다. 발생한 오류: #HY000 - 백엔드 서버와의 연결이 끊겼습니다: 네트워크 오류 (서버2: 220, 피어에 의해 연결 재설정). 마지막 종료 원인: <없음>. 마지막 오류: <비어 있음>.
```

Master Server에 varibles를 조회 해보았다.  


```
MariaDB [(none)]> show global variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.000 sec)

```

네트워크가 잠시끊겼을때 wait_timeout 28800 초가 일어나는거 같고 잠시 끊긴 네트워크가 다시 붙는데 문제가 없어보이는거같다.
그래서 다음과 같이 조치를 하였다. 새벽에는 물론 사용하는 사람이 없을거고, wait_timeout시간을 낮춰 연결을 빠르게 끊고 다시 맺으면 될꺼란 생각이 든다.

```
MariaDB [(none)]> SET GLOBAL wait_timeout=3600;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> show global variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 3600  |
+---------------+-------+
1 row in set (0.001 sec)

```

### MariaDB 공식 홈페이지 질문

나와 같은 문제가 있어보인다.

```
https://mariadb.com/kb/en/error-408595-lost-connection-to-the-master-server/+comments/4074


직역
connection_keepalive
Note: This parameter has been moved into the MaxScale core. For the current documentation, read the connection_keepalive section in the configuration guide.

Send keepalive pings to backend servers. This feature was introduced in MaxScale 2.2.0. The default value is 300 seconds starting with 2.3.2 and for older versions the feature was disabled by default. This parameter was converted into a service parameter in MaxScale 2.5.0.

번역
connection_keepalive
참고: 이 매개변수는 MaxScale 코어로 이동되었습니다. 최신 문서에 대해서는 구성 가이드의 connection_keepalive 섹션을 참조하세요.

백엔드 서버로 keepalive 핑을 보냅니다. 이 기능은 MaxScale 2.2.0에서 도입되었습니다. 기본값은 2.3.2부터 300초이며, 이전 버전에서는 기본적으로 비활성화되었습니다. 이 매개변수는 MaxScale 2.5.0에서 서비스 매개변수로 변환되었습니다.

```

Maxscale config 에도 아래와같이 옵션을 추가해주었다.

```
connection_keepalive=0s
```


```
connection_keepalive
백엔드 서버로 유휴 연결을 유지하기 위해 keepalive 핑을 보냅니다. 이 기능은 MaxScale 2.5.0에서 도입되었으며, readwritesplit 특정 기능에서 일반적인 서비스 기능으로 변경되었습니다. 이 매개변수의 기본값은 300초입니다. 이 기능을 비활성화하려면 값을 0으로 설정하세요.

keepalive 간격은 여기에 문서화된대로 지정됩니다. 명시적 단위가 제공되지 않은 경우 MaxScale 2.5에서는 초 단위로 해석됩니다. 이후 버전에서는 단위 없는 값이 거부될 수 있습니다. keepalive의 세밀성이 초 단위이기 때문에, 밀리초로 지정된 keepalive는 초보다 더 길더라도 거부될 수 있습니다.

이 매개변수 값은 각 keepalive 핑 사이의 간격을 초 단위로 나타냅니다. 만약 연결이 설정된 keepalive 간격보다 더 오랜 시간 동안 유휴 상태인 경우, 백엔드 서버로 keepalive 핑이 보내집니다.

MaxScale 2.5.21과 6.4.0부터, 클라이언트가 설정된 connection_keepalive 값보다 오랜 시간 동안 유휴 상태인 경우, keepalive 핑은 보내지지 않습니다. 이전 버전의 MaxScale은 클라이언트 상태와 관계없이 keepalive 핑을 보냈습니다.

이 매개변수는 최상위 서비스에서만 적용됩니다. 최상위 서비스란 클라이언트가 연결한 리스너가 가리키는 서비스입니다 (즉, 리스너의 service 값). 서비스가 targets 매개변수에서 다른 서비스를 정의하는 경우, 해당 서비스들에 대해서는 connection_keepalive가 사용되지 않습니다.

connection_keepalive의 값이 실행 중에 변경되면, 변경된 값이 즉시 적용됩니다.

유지하기 위해 연결 keepalive 핑은 현재 진행 중인 쿼리가 없을 때에만 수행되어야 합니다. 따라서 모든 요청과 응답은 MaxScale에 의해 추적되어야 합니다. readconnroute와 같이 결과 추적을 의존하는 라우터의 경우, 약간의 성능 저하가 발생할 수 있습니다. readwritesplit 및 schemarouter와 같이 결과 추적을 사용하는 라우터의 경우, connection_keepalive와 무관하게 성능은 동일합니다.

성능 비용을 피하고 connection keepalive 기능이 필요 없는 경우, connection_keepalive=0s로 비활성화할 수 있습니다.
```



실행시 로그를 보면 문제없이 잘 뜨는걸 볼수 있다. 경과를 지켜보도록 한다.

```
tail -f maxscale.log
2023-07-28 00:20:29   notice : Started REST API on [127.0.0.1]:8989
2023-07-28 00:20:29   notice : 'server2' sent version string '10.5.15-MariaDB-lo                      g'. Detected type: 'MariaDB', version: 10.5.15.
2023-07-28 00:20:29   notice : 'server1' sent version string '10.5.15-MariaDB-lo                      g'. Detected type: 'MariaDB', version: 10.5.15.
2023-07-28 00:20:29   notice : Server 'server2' charset: utf8_general_ci
2023-07-28 00:20:29   notice : Server 'server1' charset: utf8_general_ci
2023-07-28 00:20:29   notice : Starting a total of 1 services...
2023-07-28 00:20:29   notice : (Read-Write-Listener); Listening for connections                       at [::]:3007
2023-07-28 00:20:29   notice : Service 'Read-Write-Service' started (1/1)
2023-07-28 00:20:29   notice : MaxScale started with 24 worker threads.
2023-07-28 00:20:29   notice : Read 8 user@host entries from 'server2' for servi                      ce 'Read-Write-Service'.

```