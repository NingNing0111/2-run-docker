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
version: "3"
services:
  zookeeper:
    image: docker.io/bitnami/zookeeper:3.8
    network_mode: "bridge"
    container_name: zookeeper_1
    ports:
      - "2181:2181"
    environment:
      - TZ=Asia/Shanghai
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    restart: always
    image: docker.io/bitnami/kafka:3.4
    network_mode: "bridge"
    container_name: kafka_1
    ports:
      - "9004:9004"
    environment:
      - TZ=Asia/Shanghai
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9004
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9004 #替换成你自己的IP
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zookeeper
```
