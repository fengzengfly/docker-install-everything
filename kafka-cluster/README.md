### 0x00 创建目录

```bash
mkdir /kafka
cd /kafka
vim docker-compose.yml
```

```yml
version: "2"

services:
    # 服务名
  kafka-console-ui:
    # 容器名
    container_name: "kafka-console-ui"
    # 端口
    ports:
      - "7766:7766"
    # 持久化
    volumes:
      - ./data/kafka-ui:/app/data
      - ./logs/kafka-ui:/app/log
    # 防止读写文件有问题
    privileged: true
    user: root
    # 镜像地址
    image: "wdkang/kafka-console-ui"
    expose:
      - "7766"

  kafka1:
    container_name: kafka1
    image: 'bitnami/kafka:3.5'
    ports:
      - '19092:9092'
      - '19093:9093'
    environment:
      ### 通用配置
      # 允许使用kraft，即Kafka替代Zookeeper
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_NODE_ID=1
      # kafka角色，做broker，也要做controller
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      # 定义kafka服务端socket监听端口（Docker内部的ip地址和端口）
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      # 定义外网访问地址（宿主机ip地址和端口）ip不能是0.0.0.0
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://10.0.5.78:19092
      # 定义安全协议
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      # 集群地址
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka1:9093,2@kafka2:9093,3@kafka3:9093
      # 指定供外部使用的控制类请求信息
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      # 设置broker最大内存，和初始内存
      - KAFKA_HEAP_OPTS=-Xmx512M -Xms256M
      # 使用Kafka时的集群id，集群内的Kafka都要用这个id做初始化，生成一个UUID即可(22byte)
      - KAFKA_KRAFT_CLUSTER_ID=xYcCyHmJlIaLzLoBzVwIcP
      # 允许使用PLAINTEXT监听器，默认false，不建议在生产环境使用
      - ALLOW_PLAINTEXT_LISTENER=yes
      # 不允许自动创建主题
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # broker.id，必须唯一，且与KAFKA_CFG_NODE_ID一致
      - KAFKA_BROKER_ID=1
    volumes:
      - ./data/kafka/broker1:/bitnami/kafka:rw

  kafka2:
    container_name: kafka2
    image: 'bitnami/kafka:3.5'
    ports:
      - '29092:9092'
      - '29093:9093'
    environment:
      ### 通用配置
      # 允许使用kraft，即Kafka替代Zookeeper
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_NODE_ID=2
      # kafka角色，做broker，也要做controller
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      # 定义kafka服务端socket监听端口（Docker内部的ip地址和端口）
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      # 定义外网访问地址（宿主机ip地址和端口）ip不能是0.0.0.0
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://10.0.5.78:29092
      # 定义安全协议
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      # 集群地址
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka1:9093,2@kafka2:9093,3@kafka3:9093
      # 指定供外部使用的控制类请求信息
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      # 设置broker最大内存，和初始内存
      - KAFKA_HEAP_OPTS=-Xmx512M -Xms256M
      # 使用Kafka时的集群id，集群内的Kafka都要用这个id做初始化，生成一个UUID即可(22byte)
      - KAFKA_KRAFT_CLUSTER_ID=xYcCyHmJlIaLzLoBzVwIcP
      # 允许使用PLAINTEXT监听器，默认false，不建议在生产环境使用
      - ALLOW_PLAINTEXT_LISTENER=yes
      # 不允许自动创建主题
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # broker.id，必须唯一，且与KAFKA_CFG_NODE_ID一致
      - KAFKA_BROKER_ID=2
    volumes:
      - ./data/kafka/broker2:/bitnami/kafka:rw

  kafka3:
    container_name: kafka3
    image: 'bitnami/kafka:3.5'
    ports:
      - '39092:9092'
      - '39093:9093'
    environment:
      ### 通用配置
      # 允许使用kraft，即Kafka替代Zookeeper
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_NODE_ID=3
      # kafka角色，做broker，也要做controller
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      # 定义kafka服务端socket监听端口（Docker内部的ip地址和端口）
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      # 定义外网访问地址（宿主机ip地址和端口）ip不能是0.0.0.0
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://10.0.5.78:39092
      # 定义安全协议
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      # 集群地址
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka1:9093,2@kafka2:9093,3@kafka3:9093
      # 指定供外部使用的控制类请求信息
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      # 设置broker最大内存，和初始内存
      - KAFKA_HEAP_OPTS=-Xmx512M -Xms256M
      # 使用Kafka时的集群id，集群内的Kafka都要用这个id做初始化，生成一个UUID即可(22byte)
      - KAFKA_KRAFT_CLUSTER_ID=xYcCyHmJlIaLzLoBzVwIcP
      # 允许使用PLAINTEXT监听器，默认false，不建议在生产环境使用
      - ALLOW_PLAINTEXT_LISTENER=yes
      # 不允许自动创建主题
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      # broker.id，必须唯一，且与KAFKA_CFG_NODE_ID一致
      - KAFKA_BROKER_ID=3
    volumes:
      - ./data/kafka/broker3:/bitnami/kafka:rw
```

### 0x01 创建挂载目录文件

1. kafka数据持久化挂载，
   - `mkdir -p ./data/kafka`

2. 创建 kafka-consule 的挂载文件
   - `mkdir -p ./data/kafka-ui`
   - `mkdir -p ./logs/kafka-ui`

### 0x02 kafka启动报错 mkdir: cannot create directory '/bitnami/kafka/config': Permission denied

设置权限：`sudo chown -R 1001:1001 ./data/kafka`

这个问题 stackoverflow 和 github issues 都有提及：

[[bitnami/kafka\] Cannot create directory ‘/bitnami/kafka/config’: Permission denied · Issue #41422 · bitnami/containers (github.com)](https://github.com/bitnami/containers/issues/41422)

[kubernetes - mkdir: cannot create directory ‘/bitnami/kafka/config’: Permission denied - Stack Overflow](https://stackoverflow.com/questions/73860963/mkdir-cannot-create-directory-bitnami-kafka-config-permission-denied)

### 0x03 启动

`docker-compose up -d`