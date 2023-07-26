---
layout: post
title: Zookeeper Cluster
date: 2023-07-01 00:00 +0800
last_modified_at: 2023-07-01 00:08:25 +0800
tags: [Zookeeper Cluster]
categoie: [Zookeeper]
topic:  true
---
**Zookeeper cluseter**!
{: .message }


### Zookeeper Cluster 설치 



```
#tickTime=2000
tickTime=3000
initLimit=10

syncLimit=5

dataDir=/tmp/zookeeper

clientPort=2181
maxClientCnxns=60
autopurge.snapRetainCount=3

autopurge.purgeInterval=1

server.1=192.168.10.1:2888:3888
server.2=192.168.10.2:2888:3888
server.3=192.168.10.3:2888:3888 

```

주키퍼 클러스터 실행전에는 반드시 노드별로 주키퍼 데이타디렉토리(dataDir)에 주키퍼의 서버id가 존재해야 한다. 존재안했을시 에러가 발생하면서 실행이 되지않는점이다. 

echo 1 > /tmp/zookeeper/myid


### zookeeper 실행

bin/zookeeper-server-start.sh config/zookeeper.properties