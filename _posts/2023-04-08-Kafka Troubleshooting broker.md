---
layout: post
title: Apache Kafka broker Troubleshootiong
date: 2023-04-08 00:00 +0800
last_modified_at: 2023-04-08 00:08:25 +0800
tags: [Kafka, kafka , Kafka.cnf, Apache Kafka broker, kafka Troubleshootiong]
categoie: [kafka Troubleshootiong]
toc:  true
---
**Apache Kafka broker Troubleshootiong**!
{: .message }

# kafka Troubleshootiong


![image](https://user-images.githubusercontent.com/57931679/230721287-d975a680-1df5-421d-859e-f4886513cd27.png)

```
[2023-04-08 21:28:04,761] INFO Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2023-04-08 21:28:04,764] INFO Socket connection established, initiating session, client: /127.0.0.1:41976, server: localhost/127.0.0.1:2181 (org.apache.zookeeper.ClientCnxn)
[2023-04-08 21:28:04,769] INFO Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x1000148ef330003, negotiated timeout = 18000 (org.apache.zookeeper.ClientCnxn)
[2023-04-08 21:28:04,772] INFO [ZooKeeperClient Kafka server] Connected. (kafka.zookeeper.ZooKeeperClient)
[2023-04-08 21:28:04,842] INFO [feature-zk-node-event-process-thread]: Starting (kafka.server.FinalizedFeatureChangeListener$ChangeNotificationProcessorThread)
[2023-04-08 21:28:04,967] INFO Updated cache from existing <empty> to latest FinalizedFeaturesAndEpoch(features=Features{}, epoch=0). (kafka.server.FinalizedFeatureCache)
[2023-04-08 21:28:04,971] INFO Cluster ID = Ku66UbL0QvWZLunouO9gHw (kafka.server.KafkaServer)
[2023-04-08 21:28:04,980] ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.common.InconsistentBrokerIdException: Configured broker.id 1 doesn't match stored broker.id Some(0) in meta.properties. If you moved your data, make sure your configured broker.id matches. If you intend to create a new broker, you should remove all data in your data directories (log.dirs).
        at kafka.server.KafkaServer.getOrGenerateBrokerId(KafkaServer.scala:793)
        at kafka.server.KafkaServer.startup(KafkaServer.scala:221)
        at kafka.Kafka$.main(Kafka.scala:109)
        at kafka.Kafka.main(Kafka.scala)
[2023-04-08 21:28:04,981] INFO shutting down (kafka.server.KafkaServer)
[2023-04-08 21:28:04,983] INFO [feature-zk-node-event-process-thread]: Shutting down (kafka.server.FinalizedFeatureChangeListener$ChangeNotificationProcessorThread)
[2023-04-08 21:28:04,984] INFO [feature-zk-node-event-process-thread]: Stopped (kafka.server.FinalizedFeatureChangeListener$ChangeNotificationProcessorThread)
[2023-04-08 21:28:04,984] INFO [feature-zk-node-event-process-thread]: Shutdown completed (kafka.server.FinalizedFeatureChangeListener$ChangeNotificationProcessorThread)
[2023-04-08 21:28:04,986] INFO [ZooKeeperClient Kafka server] Closing. (kafka.zookeeper.ZooKeeperClient)
[2023-04-08 21:28:05,092] INFO Session: 0x1000148ef330003 closed (org.apache.zookeeper.ZooKeeper)
[2023-04-08 21:28:05,092] INFO EventThread shut down for session: 0x1000148ef330003 (org.apache.zookeeper.ClientCnxn)
[2023-04-08 21:28:05,099] INFO [ZooKeeperClient Kafka server] Closed. (kafka.zookeeper.ZooKeeperClient)
[2023-04-08 21:28:05,107] INFO App info kafka.server for 1 unregistered (org.apache.kafka.common.utils.AppInfoParser)
[2023-04-08 21:28:05,108] INFO shut down completed (kafka.server.KafkaServer)
[2023-04-08 21:28:05,108] ERROR Exiting Kafka. (kafka.Kafka$)
[2023-04-08 21:28:05,111] INFO shutting down (kafka.server.KafkaServer)

```

위와 같이 에러가 발생했으나, 문제 원인은 broker id 가 0번 에 데이터가 log.dirs 파일이 생겨있다가 broker id 가 1으로 변경하여 에러가 발생한것이다. 해결 방법은 broker 1번으로 변경하려 할 경우 log.dirs에 있는 데이터를 삭제한후에 변경하면 된다.