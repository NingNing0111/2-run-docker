# 使用 Docker 安装常见的环境

## 数据库

### MySQL

```yml
docker run --name mysql \
--restart=always \
-p 3306:3306 \
-v /data/mysql/log:/var/log/mysql \
-v /data/mysql/data:/var/lib/mysql \
-v /data/mysql/conf.d:/etc/mysql/conf.d \
-v /data/mysql/my.cnf:/etc/my.cnf \
-e MYSQL_ROOT_PASSWORD=rootPassword \
-e MYSQL_USER=user1 \
-e MYSQL_PASSWORD=password1 \
-e MYSQL_DATABASE=databaseName \
-d mysql:5.7
```

### PostgreSQL

```yml
$ docker run --name postgres \
--restart=always \
-p 5432:5432 \
-e POSTGRES_PASSWORD=passowrd \
-e PGDATA=/var/lib/postgresql/data/pgdata \
-d postgres:latest
```

### Redis

```yml
docker run --name redis \
--restart always \
--privileged=true \
--appendonly yes \
--requirepass "password" \
-p 16379:6379  \
-v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data:rw \
-d redis redis-server /etc/redis/redis.conf \
```

## 微服务组件

### 注册中心

#### Zookeeper

```yml
docker run  --name zookeeper \
--privileged=true \
-p 2181:2181  \
-v ./zookeeper/data:/data \
-v ./zookeeper/conf:/conf \
-v ./zookeeper/logs:/datalog \
-d zookeeper:3.5.7
```

&emsp;`./zookeeper/conf/zoo.cfg`:

```ini
dataDir=/data  # 保存zookeeper中的数据
clientPort=2181 # 客户端连接端口，通常不做修改
dataLogDir=/datalog
tickTime=2000  # 通信心跳时间
initLimit=5    # LF(leader - follower)初始通信时限
syncLimit=2    # LF 同步通信时限
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
standaloneEnabled=true
admin.enableServer=true
server.1=localhost:2888:3888;2181
```

### 消息队列

#### kafka

```yml
docker run --name kafka \
--restart always \
-p 29092:29092 \
-v data:/bitnami/kafka \
-e KAFKA_BROKER_ID=1 \
-e KAFKA_ZOOKEEPER_CONNECT=localhost:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:29092 \
-d apache/kafka:latest
```

## Docker Compose

### Zookeeper-Kafka

```yml
version: "2"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.4
    container_name: zookeeper
    hostname: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181

  kafka:
    image: confluentinc/cp-kafka:7.4.4
    container_name: kafka
    hostname: kafka
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
      - 9997:9997
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9997
      KAFKA_JMX_HOSTNAME: kafka

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    depends_on:
      - kafka
    ports:
      - 8080:8080
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
      KAFKA_CLUSTERS_0_METRICS_PORT: 9997
      DYNAMIC_CONFIG_ENABLED: "true"
```
