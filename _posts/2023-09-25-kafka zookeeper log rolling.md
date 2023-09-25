---
layout: post
title: Kafka broker log rolling
date: 2023-09-25 00:00 +0800
last_modified_at: 2023-09-25 00:08:25 +0800
tags: [kafka broker log rolling ]
categoie: [kafka broker log rolling]
topic:  true
---
**kafka log rolling**!
{: .message }

### System Log

보통 System에서 Log는 System의 상황 파악을 위해 기록하는 정보이다. Kafka에서도 Broker에서 이런 Log를 기록하고 있다. server, connect, controller, user 등 다양한 Log를 기록 하고 있다.
이런 Log 설정은 conf/log4j.proerties에서 확인 할 수 있다.   

```
$ cat /kafka/conf/log4j.properties

...
log4j.appender.kafkaAppender=org.apache.log4j.DailyRollingFileAppender
log4j.appender.kafkaAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.kafkaAppender.File=${kafka.logs.dir}/server.log                                                                 
log4j.appender.kafkaAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.kafkaAppender.layout.ConversionPattern=[%d] %p %m (%c)%n
...

```

위 설정 값은 기본값이며, default값으로 두면 오래된 Log를 처리하는 로직이 없어 Log가 큰용량을 차지 할 수 있다.

이번 포스팅은 Log파일을 정리하는 방법을 하려한다.

RollingFileAppender

기본 DaillyRollingFileAppender가 아닌 RollingFileAppender로 설정을 변경한다.

MaxFileSize와 MaxBackupIndex 설정값으로 오래된 Log를 삭제 할 수 있다.

```
log4j.appender.kafkaAppender=org.apache.log4j.RollingFileAppender
log4j.appender.kafkaAppender.File=${kafka.logs.dir}/server.log                                                                 
log4j.appender.kafkaAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.kafkaAppender.layout.ConversionPattern=[%d] %p %m (%c)%n
log4j.appender.kafkaAppender.MaxFileSize=128MB
log4j.appender.kafkaAppender.MaxBackupIndex=10
```


## Crontab

두 번째는 crontab을 활용하여 오래된 log 파일을 지워준다.

crontab을 활용하면 특정 작업을 schedule을 통해 실행할 수 있다.

먼저 오랜된 log파일을 지워주는 script를 생성한다.

```
$ vi kafka_rm_logs.sh

#!/bin/sh

/usr/bin/find /kafka/logs/ -type f -mtime +3 -exec rm {} +

```

Kafka Log 폴더에서 3일 이상이 된 파일을 삭제하는 script이다.  

해당 script를 crontab에 등록한다.  

```
# crontab 확인
$ crontab -l

# crontab 수정
$ crontab -e

# kafka 3 day older log delete
00 00 * * * kafka_rm_logs.sh
```

매일 자정에 kafka_rm_logs.sh 파일이 실행되며 오래된 log를 삭제한다.  