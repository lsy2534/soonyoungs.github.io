---
layout: post
title: MariaDB 운영 Relay Log
date: 2023-08-04 00:00 +0800
last_modified_at: 2023-08-04 00:08:25 +0800
tags: [MariaDB 운영환경 로그 ]
categoie: [MariaDB 운영환경 로그]
topic:  true
---
**MariaDB 운영 log**!
{: .message }

### Relay Log

```
MariaDB의 릴레이 로그 문제는 손상된 마스터 바이너리 로그, 잘못된 복제본 호스트 이름 또는 마스터와 동기화되지 않은 복제본과 같은 다양한 문제로 인해 발생할 수 있습니다. 이러한 문제에 대한 해결 방법으로는 SET GLOBAL sqlslaveskip_counter를 사용하여 마스터 바이너리 로그에서 하나의 이벤트를 건너뛰고, 바이너리 로그 및 릴레이 로그 파일을 읽을 때 할당된 인메모리 캐시 크기를 늘리고,복제본에서 --relay-log=mysqld-relay-bin 옵션을 사용하고, 마스터 바이너리 로그의 손상 여부를 확인하는 것 등이 있습니다. 또한 복제본이 마스터와 동기화되지 않아 문제가 발생하는 경우 복제본을 중지하고 릴레이 로그를 삭제한 다음 복제본을 다시 시작하여 복제본과 마스터를 다시 동기화해야 할 수 있습니다.

기본 옵션으로 아래와 같이 자동 삭제 여부가 켜져있는걸로 알고있습니다.
```

MariaDB [(none)] > set global relay_log_purge = on;
MariaDB [(none)] > show global variables like 'relay_log_purge';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| relay_log_purge | ON       |
+-----------------+----------+

my.cnf
# 1 -> ON / 0 -> OFF
relay_log_purge = 1
```

