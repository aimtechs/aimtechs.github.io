---
published: true
title: "kafka tutorial"
layout: post
author: jihwan, hwang
category: apache
tags:
- kafka
- apache-kafka
---


# intro

2016년 12월 끝자락 회의 중 [kafka](http://kafka.apache.org/) 가 언급 되었고, 이를 다뤄 보게 되었다.
우선 결과 부터 언급하자면, application 수준에서 충족되어야 하는 기능이 [kafka](http://kafka.apache.org/) 에서는 지원하지 않아 중단한다.
다만, 몇 일간 [kafka](http://kafka.apache.org/) 를 다루면서 알게된 사용법과 application 코드의 자취를 남기고자 한다.

**apache kafka** 를 검색하면 많은 내용이 있으므로, 이를 통해 개념을 잡으면 되겠다.

여기서는 테스트 한 [kafka](http://kafka.apache.org/) 명령, property, java code 정도만 나열 한다.

[kafka download](http://kafka.apache.org/downloads)

[kafka documentation](http://kafka.apache.org/documentation/)

[kafka quicstart](http://kafka.apache.org/quickstart)

참고로, [kafka](http://kafka.apache.org/) 는 분산 클러스터링 구성으로 운영되는데 이 조율을 zookeeper 에서 담당하고 있다. zookeeper 역시 분산 클러스터링 구성을 해야 한다. 그러나 여기서는, zookeeper 를 single instance 로 사용하겠다. 그리고, [kafka](http://kafka.apache.org/) 내장 zookeeper 를 사용한다.


테스트는 아래와 같이 두가지를 하였다.
- 단일노드 테스트 : download 받은 [kafka](http://kafka.apache.org/) 를 직접 운영하여 테스트 진행
- 클러스터 테스트 : [docker](https://www.docker.com/) 를 이용하여 [kafka](http://kafka.apache.org/) 클러스터를 구성하여 테스트 진행

--------------------------------------------------

## config/server.properties 프로퍼티 변경 항목

- delete.topic.enable=true


--------------------------------------------------

# 단일 노드 테스트

## zookeeper 실행

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties &

ps -ef | grep zookeeper
```

## kafka 실행

```bash
bin/kafka-server-start.sh config/server.properties &

ps -ef | grep kafkaServer
```

kafka가 실행 되면, kafka 실행에 필요한 정보 파일이 생성된다.

- cleaner-offset-checkpoint
- meta.properties
- recovery-point-offset-checkpoint
- replication-offset-checkpoint


[broker config](http://kafka.apache.org/documentation/#configuration)


## zookeeper and kafka server log

- controller.log
- kafka-authorizer.log
- kafka-request.log
- log-cleaner.log
- server.log
- state-change.log
- zookeeper-gc.log


## topic

```bash
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 2 --topic topic1

bin/kafka-topics.sh --list --zookeeper localhost:2181

bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic topic1

bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic topic1 --partitions 3
```




--------------------------------------------------

# 클러스터 테스트

[reference](http://wurstmeister.github.io/kafka-docker/)


```bash
docker pull wurstmeister/kafka

docker images

docker-compose up -d

docker-compose scale kafka=3

docker-compose stop

docker-compose rm -f
```


```bash
docker exec -it dockerkafka_kafka_1 bash
cd /opt/kafka
bin/kafka-topics.sh --list --zookeeper 192.168.100.213:2181
bin/kafka-topics.sh --describe --zookeeper 192.168.100.213:2181 --topic topic1
```

---------------------------------------------

# 테스트 코드 링크

[go](https://github.com/aimtechs/kafka-test)
