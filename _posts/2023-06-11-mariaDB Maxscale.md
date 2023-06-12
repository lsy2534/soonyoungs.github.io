---
layout: post
title: MariaDB Maxscale
date: 2023-06-12 00:00 +0800
last_modified_at: 2023-06-12 00:08:25 +0800
tags: [CentOS 8 Stream, MariaDB Cluster, Maxscale]
categoie: [MariaBD]
toc:  true
---
**MariaDB MaxScale**!
{: .message }

Mariadb MaxScale은 데이터베이스 명령문을 하나 이상의 데이터베이스 서버로 전달하는 데이터베이스 프록시이다.
전달은 데이터베이스 명령문의 의미론적 이해와 데이터 베이스 백엔드 클러스터 내의 서버 역할에 기반한 규칙을 사용하여 수행한다.
애플리케이션에 투명하게 로드밸런싱 및 고가용성 기능을 제공하도록 설계되어서 다양한 프로토콜 및 라이팅 접근 방식을 지원하는 플러그인 구성 요소와 함께 확장 가능하고 유연한 아키텍처를 갖추고 있다.

### 구성도

![Maxcale 구성도](https://github.com/lsy2534/soonyoungs.github.io/assets/57931679/84886f33-d110-4bbe-a7e9-18b4c2b72d54)

## 구성요소

* MariaDB Server Master  
* MariaDB server Slave  

> 두 서버는 MariaDB GTID를 기반으로 Semi-Sync Replication Set을 구성합니다.

* MaxScale 
> auto_failover, auto_rejoin

주의 : Maxscale의 auto_failover / outo_rejoin등은 MariaDB 기반의 GTID만 지원합니다.
```
MariaDB GTID(Global Transaction Identifier)는 MariaDB 데이터베이스에서 복제된 트랜잭션을 고유하게 식별하는 전역적인 식별자입니다. GTID는 마스터 서버와 슬레이브 서버 간의 복제를 관리하는 데 사용됩니다.

GTID는 각각의 트랜잭션에 고유한 식별자를 부여함으로써 데이터의 일관성과 신뢰성을 유지합니다. GTID는 마스터 서버에서 생성되며, 슬레이브 서버가 이를 이용하여 마스터 서버로부터 전송받을 트랜잭션을 추적하고 복제합니다. 이를 통해 복제 과정에서 발생하는 데이터의 불일치나 중복 복제 문제를 방지할 수 있습니다.

GTID는 일련번호와 서버 식별자로 구성됩니다. 일련번호는 트랜잭션의 순서를 나타내며, 서버 식별자는 해당 트랜잭션이 생성된 서버를 식별합니다. 이렇게 구성된 GTID는 복제 과정에서 트랜잭션을 고유하게 식별하고, 마스터 서버와 슬레이브 서버 간에 복제 상태를 동기화하는 데 사용됩니다.

MariaDB GTID는 데이터베이스의 복제 프로세스를 보다 효율적이고 안정적으로 관리할 수 있는 기능을 제공하며, 데이터의 일관성과 신뢰성을 보장하는 데 도움을 줍니다.
```

## 구성 파일
OS : Red Hat Enterprise Linux 8.4

Maxscale 구성 필요 파일 및 라이브러리

maxscale-23.02.2.rhel.8-2.tar.gz  
unixODBC-2.3.11-1.rh.x86_64.rpm  
libatomic-8.5.0-2.el8.x86_64.rpm  

실습 예제에서는 아래와 같은 디렉토리 구조에서 실습 하겠습니다. 
```
/app
```

## MaxScale tar 압축 해제 및 이동

tar zxvf maxscale-23.02.2.rhel.8-2.tar.gz
mv maxscale-23.02.2.rhel.8-2 /app/maxscale
cd /app/maxscale

## 라이브러리 설치
sudo rpm -i unixODBC-2.3.11-1.rh.x86_64.rpm
sudo rpm -i libatomic-8.5.0-2.el8.x86_64.rpm

## 관련 config 생성
```
$ vi ./etc/maxscale.cnf

[maxscale]
threads=auto

[server1]
type=server
address=MasterIP
port=3306
protocol=MariaDBBackend

[server2]
type=server
address=SlaveIP
port=3306
protocol=MariaDBBackend

[MariaDB-Monitor]
type=monitor
module=mariadbmon
servers=server1,server2
user=monitor_user
password=monitor_user
monitor_interval=2s
auto_failover=true
auto_rejoin=true
enforce_read_only_slaves=1
handle_events=true

[Read-Write-Service]
type=service
router=readwritesplit
servers=server1,server2
user=maxscale
password=maxscale
use_sql_variables_in=master
enable_root_user=1

[Read-Write-Listener]
type=listener
service=Read-Write-Service
protocol=MariaDBClient
port=4006

```



### MariaDB Master, Slave Monitoring 계정 생성

* MariaDB Master Server , MariaDB Slave Server 접속 후 계정 생성
```
CREATE USER 'monitor_user'@'%' IDENTIFIED BY 'monitor_user';
GRANT REPLICATION CLIENT, FILE, SUPER, RELOAD, PROCESS, SHOW DATABASES, EVENT ON *.* TO 'monitor_user'@'%';
```
### MariaDB Master, Slave Read-Write Service 계정 생성

* MariaDB Master Server , MariaDB Slave Server 접속 후 계정 생성
```
CREATE USER 'maxscale'@'%' IDENTIFIED BY 'maxscale';
GRANT ALL PRIVILEGES ON *.* TO 'maxscale'@'%';
FLUSH PRIVILEGES;
```

### MaxScale 실행
```
$ bin/maxscale -d --basedir=.

## 실행 로그

2023-06-12 23:41:17   notice : /home/rwk/maxscale/maxscale-23.02.2.rhel.8/etc/maxscale.cnf.d does not exist, not reading.
2023-06-12 23:41:17   notice : Module 'mariadbmon' loaded from './lib64/maxscale/libmariadbmon.so'.
2023-06-12 23:41:17   notice : Module 'readwritesplit' loaded from './lib64/maxscale/libreadwritesplit.so'.
2023-06-12 23:41:17   notice : Using up to 560.31MiB of memory for query classifier cache
2023-06-12 23:41:17   notice : syslog logging is disabled.
2023-06-12 23:41:17   notice : maxlog logging is enabled.
2023-06-12 23:41:17   notice : Host: 'localhost.localdomain' OS: Linux@4.18.0-305.el8.x86_64, #1 SMP Thu Apr 29 08:54:30 EDT 2021, x86_64 with 8 processor cores (8.00 available).
2023-06-12 23:41:17   notice : Total main memory: 3.65GiB (3.65GiB usable).
2023-06-12 23:41:17   notice : MariaDB MaxScale 23.02.2 started (Commit: c5fce265b3cde0b4f54e6e1d0e4ada15226cfdbd)
2023-06-12 23:41:17   notice : MaxScale is running in process 38602
2023-06-12 23:41:17   notice : Configuration file: /home/rwk/maxscale/maxscale-23.02.2.rhel.8/etc/maxscale.cnf
2023-06-12 23:41:17   notice : Log directory: ./var/log/maxscale
2023-06-12 23:41:17   notice : Data directory: ./var/lib/maxscale
2023-06-12 23:41:17   notice : Module directory: ./lib64/maxscale
2023-06-12 23:41:17   notice : Service cache: ./var/cache/maxscale
2023-06-12 23:41:17   error  : Failed to remove the data directory /data33559 of MaxScale due to 2, No such file or directory.
2023-06-12 23:41:17   notice : Module 'qc_sqlite' loaded from './lib64/maxscale/libqc_sqlite.so'.
2023-06-12 23:41:17   notice : Query classification results are cached and reused. Memory used per thread: 70.04MiB
2023-06-12 23:41:17   notice : Password encryption key file './var/lib/maxscale/.secrets' not found, using configured passwords as plaintext.
2023-06-12 23:41:17   notice : Using HS256 for JWT signatures
2023-06-12 23:41:17   warning: The MaxScale GUI is enabled but encryption for the REST API is not enabled, the GUI will not be enabled. Configure `admin_ssl_key` and `admin_ssl_cert` to enable HTTPS or add `admin_secure_gui=false` to allow use of the GUI without encryption.
2023-06-12 23:41:17   notice : Started REST API on [127.0.0.1]:8989
2023-06-12 23:41:17   notice : 'server2' sent version string '10.5.15-MariaDB'. Detected type: 'MariaDB', version: 10.5.15.
2023-06-12 23:41:17   notice : 'server1' sent version string '10.5.15-MariaDB-log'. Detected type: 'MariaDB', version: 10.5.15.
2023-06-12 23:41:17   notice : Server 'server1' charset: utf8_general_ci
2023-06-12 23:41:17   notice : Server 'server2' charset: utf8_general_ci
2023-06-12 23:41:17   warning: [mariadbmon] 'server1' is a better primary candidate than the current primary 'server2'. Primary will change when 'server2' is no longer a valid primary.
2023-06-12 23:41:17   warning: [mariadbmon] Automatic rejoin was not attempted on server 'server1' even though it is a valid candidate. Will keep retrying with this message suppressed for all servers. Errors:
'server2' does not have a valid gtid_binlog_pos.
2023-06-12 23:41:17   notice : Starting a total of 1 services...
2023-06-12 23:41:17   notice : (Read-Write-Listener); Listening for connections at [::]:4006
2023-06-12 23:41:17   notice : Service 'Read-Write-Service' started (1/1)
2023-06-12 23:41:17   notice : MaxScale started with 8 worker threads.
2023-06-12 23:41:18   notice : Read 8 user@host entries from 'server2' for service 'Read-Write-Service'.
```


### MaxScale 상태 확인
```
$ bin/maxctrl list servers
┌─────────┬───────────────┬──────┬─────────────┬─────────────────┬────────┬─────────────────┐
│ Server  │ Address       │ Port │ Connections │ State           │ GTID   │ Monitor         │
├─────────┼───────────────┼──────┼─────────────┼─────────────────┼────────┼─────────────────┤
│ server1 │ 192.168.0.210 │ 3306 │ 0           │ Running         │ 0-1-74 │ MariaDB-Monitor │
├─────────┼───────────────┼──────┼─────────────┼─────────────────┼────────┼─────────────────┤
│ server2 │ 192.168.0.211 │ 3306 │ 0           │ Master, Running │ 0-1-74 │ MariaDB-Monitor │
└─────────┴───────────────┴──────┴─────────────┴─────────────────┴────────┴─────────────────┘

```

엇.. 위로그는 server1을 죽였다가 살렸더니 server2(slave)서버 로는 이동이 되었지만 실제 Master 서버로는 이동이 되지 않은듯 하다.
이럴땐, 다시 동기화를 해주고 server1번 부터 다시 재기동을 해줘야 다시 server1이 Master Server를 가져가는거 같습니다..
 테스트는 이어서 진행하도록 하겠습니다.  