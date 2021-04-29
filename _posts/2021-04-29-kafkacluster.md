---
title: "kafka Cluster 환경"
layout: post
date: '2021-04-29 14:18:52'
author: jsh
categories: Kafka
---

### 설치 모듈
> * docker, docker-compose, kafka-stack-docker-compose

### docker, docker-compose 설치
```groovy
# docker 설치
[root@tivlxdkkfs-kafka /]# yum install docker -y

# 부팅 시 자동실행 활성화
[root@tivlxdkkfs-kafka /]# systemctl enabled docker.service

# docker 시작
[root@tivlxdkkfs-kafka /]# systemctl start docker.service

# docker-compose 설치
[root@tivlxdkkfs-kafka /]# sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# docker-compose 실행 권한
[root@tivlxdkkfs-kafka /]# sudo chmod +x /usr/local/bin/docker-compose
 
# 심볼릭 링크가 필요한 경우는 아래 명령어 실행
[root@tivlxdkkfs-kafka /]# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# docker-compose 설치 확인
[root@tivlxdkkfs-kafka /]# docker-compose --version
docker-compose version 1.29.1, build c34c88b2
```

### kafka-stack-docker-compose 구성
> * Zookeeper version: 3.4.9
> * Kafka version: 2.5.0 (Confluent 5.5.1)
> * Kafka Schema Registry: Confluent 5.5.1
> * Kafka Schema Registry UI: 0.9.5
> * Kafka Rest Proxy: Confluent 5.5.1
> * Kafka Topics UI: 0.9.4
> * Kafka Connect: Confluent 5.5.1
> * Kafka Connect UI: 0.9.7
> * ksqlDB Server: Confluent 6.1.1
> * Zoonavigator: 0.8.0

### kafka-stack-docker-compose 설치 및 실행
```groovy
# git 설치
[root@tivlxdkkfs-kafka /]# yum install git -y

# git clone
# kafka-stack-docker-compose 설치 할 경로로 이동
[root@tivlxdkkfs-kafka /]# cd /app/docker/ 
[root@tivlxdkkfs-kafka docker]# git clone https://github.com/suhojang/kafka-stack-docker-compose.git

[root@tivlxdkkfs-kafka docker]# ls -al
total 8
drwxr-xr-x.  4 root root   58 Apr 28 14:43 .
drwxr-xr-x.  3 root root   20 Apr 28 14:23 ..
drwxr-xr-x.  6 root root 4096 Apr 29 09:05 kafka-stack-docker-compose
drwxr-xr-x. 12 root root 4096 Apr 28 14:59 librdkafka

[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker-compose -f zk-multiple-kafka-multiple.yml up
Creating network "kafka-stack-docker-compose_default" with the default driver
Creating kafka-stack-docker-compose_zoo1_1 ... done
Creating kafka-stack-docker-compose_zoo2_1 ... done
Creating kafka-stack-docker-compose_zoo3_1 ... done
Creating kafka-stack-docker-compose_kafka2_1           ... done
Creating kafka-stack-docker-compose_kafka3_1           ... done
Creating kafka-stack-docker-compose_zoonavigator-api_1 ... done
Creating kafka-stack-docker-compose_kafka1_1           ... done
Creating kafka-stack-docker-compose_zoonavigator-web_1      ... done
Creating kafka-stack-docker-compose_ksql-server_1           ... done
Creating kafka-stack-docker-compose_kafka-schema-registry_1 ... done
Creating kafka-stack-docker-compose_kafka-rest-proxy_1      ... done
Creating kafka-stack-docker-compose_schema-registry-ui_1    ... done
Creating kafka-stack-docker-compose_kafka-topics-ui_1       ... done
Creating kafka-stack-docker-compose_kafka-connect_1         ... done
Creating kafka-stack-docker-compose_kafka-connect-ui_1      ... done
Attaching to kafka-stack-docker-compose_zoo1_1, kafka-stack-docker-compose_zoo2_1, kafka-stack-docker-compose_zoo3_1, kafka-stack-docker-compose_zoonavigator-api_1, kafka-stack-docker-compose_kafka2_1, kafka-stack-docker-compose_kafka3_1, kafka-stack-docker-compose_kafka1_1, kafka-stack-docker-compose_ksql-server_1, kafka-stack-docker-compose_kafka-schema-registry_1, kafka-stack-docker-compose_zoonavigator-web_1, kafka-stack-docker-compose_kafka-rest-proxy_1, kafka-stack-docker-compose_schema-registry-ui_1, kafka-stack-docker-compose_kafka-topics-ui_1, kafka-stack-docker-compose_kafka-connect_1, kafka-stack-docker-compose_kafka-connect-ui_1
....

# docker kafka container 확인
[root@tivlxdkkfs-kafka kafka-stack-docker-compose]# docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED              STATUS                        PORTS                                                  NAMES
c9a1cf63fa68        landoop/kafka-connect-ui:0.9.4          "/run.sh"                About a minute ago   Up About a minute             0.0.0.0:8003->8000/tcp                                 kafka-stack-docker-compose_kafka-connect-ui_1
26416756e1d9        confluentinc/cp-kafka-connect:5.2.1     "/etc/confluent/do..."   About a minute ago   Up About a minute             0.0.0.0:8083->8083/tcp, 9092/tcp                       kafka-stack-docker-compose_kafka-connect_1
e18541e60872        landoop/kafka-topics-ui:0.9.4           "/run.sh"                About a minute ago   Up About a minute             0.0.0.0:8000->8000/tcp                                 kafka-stack-docker-compose_kafka-topics-ui_1
1656400927c3        landoop/schema-registry-ui:0.9.4        "/run.sh"                About a minute ago   Up About a minute             0.0.0.0:8001->8000/tcp                                 kafka-stack-docker-compose_schema-registry-ui_1
427f794a6ef8        confluentinc/cp-kafka-rest:5.2.1        "/etc/confluent/do..."   About a minute ago   Up About a minute             0.0.0.0:8082->8082/tcp                                 kafka-stack-docker-compose_kafka-rest-proxy_1
389cc682962c        confluentinc/cp-schema-registry:5.5.1   "/etc/confluent/do..."   About a minute ago   Up About a minute             0.0.0.0:8081->8081/tcp                                 kafka-stack-docker-compose_kafka-schema-registry_1
f74747dbace2        confluentinc/cp-ksql-server:5.2.1       "/etc/confluent/do..."   About a minute ago   Up About a minute             0.0.0.0:8088->8088/tcp                                 kafka-stack-docker-compose_ksql-server_1
e9c3a46a0393        elkozmon/zoonavigator-web:0.5.1         "./run.sh"               About a minute ago   Up About a minute (healthy)   80/tcp, 0.0.0.0:8004->8000/tcp                         kafka-stack-docker-compose_zoonavigator-web_1
4ee0523afb8e        confluentinc/cp-kafka:5.5.1             "/etc/confluent/do..."   About a minute ago   Up About a minute             9092/tcp, 0.0.0.0:9094->9094/tcp                       kafka-stack-docker-compose_kafka3_1
1edd5402d15d        elkozmon/zoonavigator-api:0.5.1         "./run.sh"               About a minute ago   Up About a minute (healthy)   9000/tcp                                               kafka-stack-docker-compose_zoonavigator-api_1
32f5ab448f41        confluentinc/cp-kafka:5.5.1             "/etc/confluent/do..."   About a minute ago   Up About a minute             0.0.0.0:9092->9092/tcp                                 kafka-stack-docker-compose_kafka1_1
65091e1720b7        confluentinc/cp-kafka:5.5.1             "/etc/confluent/do..."   About a minute ago   Up About a minute             9092/tcp, 0.0.0.0:9093->9093/tcp                       kafka-stack-docker-compose_kafka2_1
a2cdfb4167c2        zookeeper:3.4.9                         "/docker-entrypoin..."   About a minute ago   Up About a minute             2181/tcp, 2888/tcp, 3888/tcp, 0.0.0.0:2183->2183/tcp   kafka-stack-docker-compose_zoo3_1
bdab2952acac        zookeeper:3.4.9                         "/docker-entrypoin..."   About a minute ago   Up About a minute             2181/tcp, 2888/tcp, 3888/tcp, 0.0.0.0:2182->2182/tcp   kafka-stack-docker-compose_zoo2_1
13277244e7e6        zookeeper:3.4.9                         "/docker-entrypoin..."   About a minute ago   Up About a minute             2888/tcp, 0.0.0.0:2181->2181/tcp, 3888/tcp             kafka-stack-docker-compose_zoo1_1
```

- 구성은 아래와 같다.   
![/assets/kafka-docker-cluster.png](/assets/kafka-docker-cluster.png)
  
※ docker selinux 이슈
- file system permission denied
- 해결방법
> chcon -Rt svirt_sandbox_file_t [path]

※ Cluster 구성을 위한 다른 방법
- [https://team-platform.tistory.com/13?category=829378](https://team-platform.tistory.com/13?category=829378)
