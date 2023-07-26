---
layout: post
title: MariaDB Maxscale 테스트
date: 2023-06-12 00:00 +0800
last_modified_at: 2023-06-12 00:08:25 +0800
tags: [CentOS 8 Stream, MariaDB Cluster, Maxscale]
categoie: [MariaBD]
toc:  true
---
**MariaDB MaxScale**!
{: .message }


### MaxScale 실행

```
$ bin/maxscale -d --basedir=. debug=enable-statement-logging 

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

전 시간에는 Maxscale 구동시 위와 같은 Maxscale 구성의 상태를 확인 할수 있었다.  
하지만 이번 시간에는 오류와 Maxscale에 대한 상태 를 좀 확인하고 master와 slave간에 동기화 방법에 대해서 설명하고자 한다.

일단 maxscale 실행시 debug를 통해서 어떤 로그가 발생하는지를 확인하기 위해 실행 옵션을 추가한다.

```
$ bin/maxscale -d --basedir=. debug=enable-statement-logging 




2023-06-12 23:41:17   warning: The MaxScale GUI is enabled but encryption for the REST API is not enabled, the GUI will not be enabled. Configure `admin_ssl_key` and `admin_ssl_cert` to enable HTTPS or add `admin_secure_gui=false` to allow use of the GUI without encryption.



maxscale.cnf 
admin_secure_gui=false 

GUI 사용에 대한 옵션을 false로 지정한다.


두 상태의 동기화를 위해 슬레이브 서버에서 현지 gtid의 동기화를 기동한다.

STOP SLAVE;
CHANGE MASTER TO
   MASTER_USE_GTID = current_pos;
START SLAVE;



$ bin/maxctrl list servers
┌─────────┬───────────────┬──────┬─────────────┬─────────────────┬────────┬─────────────────┐
│ Server  │ Address       │ Port │ Connections │ State           │ GTID   │ Monitor         │
├─────────┼───────────────┼──────┼─────────────┼─────────────────┼────────┼─────────────────┤
│ server1 │ 192.168.0.210 │ 3306 │ 0           │ Master, Running │ 0-1-74 │ MariaDB-Monitor │
├─────────┼───────────────┼──────┼─────────────┼─────────────────┼────────┼─────────────────┤
│ server2 │ 192.168.0.211 │ 3306 │ 0           │ Slave, Running  │ 0-1-74 │ MariaDB-Monitor │
└─────────┴───────────────┴──────┴─────────────┴─────────────────┴────────┴─────────────────┘  

````

위와 같이 서로 다른 GTID를 실수로 동기화 하지 못할경우 프로젝트에서 failover시 자동으로 switchover가 동작하지 못하는 경우가 발생한다.
gtid를 동기화 하여 문제를 해결할수 있다.
