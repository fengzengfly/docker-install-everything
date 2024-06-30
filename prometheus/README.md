## 0x01 创建目录

找一个你喜欢的地方，创建项目根目录

example:

```shell
[root@demo-78 ~]#  mkdir /data/prometheus
```

## 0x02 创建配置文件

进入到项目根目录：

```shell
[root@demo-78 ~]#  cd /data/prometheus
```

需要新建三个文件，分别是`docker-compose.yml`、`prometheus.yml`、`node_down.yml`，详细配置如下

> 在以下配置中，除了`docker-compose.yml`中的端口映射配置以外，所有你能看到的关于`host`、`port`、`username`、`passowrd`的配置，都修改成你自己的，顺便注意一下`docker-compose.yml`配置中关于文件挂载的路劲问题。

`prometheus.yml`配置如下：

```yaml
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['10.0.5.78:9093']
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['10.0.5.78:9094']

  - job_name: 'redis'
    static_configs:
     - targets: ['10.0.5.78:9121']
       labels:
         instance: redis

  - job_name: 'node'
    scrape_interval: 8s
    static_configs:
     - targets: ['10.0.5.78:9100']
       labels:
         instance: node

  - job_name: 'cadvisor'
    static_configs:
     - targets: ['10.0.5.78:8088']
       labels:
         instance: cadvisori
         
  #基于文件自动加载新监控任务
  - job_name: 'file_ds'
    file_sd_configs:
    - files: ['/etc/prometheus/reload/*.yml']
      refresh_interval: 5s
```

`docker-compose.yml`配置如下：

```yaml
version: '3'

networks:
    monitor:
        driver: bridge

services:
    prometheus:
        image: prom/prometheus
        container_name: prometheus
        hostname: prometheus
        restart: always
        volumes:
            - /prometheus/reload:/etc/prometheus/reload
            - /prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
            - /prometheus/node_down.yml:/etc/prometheus/node_down.yml
        ports:
            - "9094:9090"
        networks:
            - monitor

    grafana:
        image: grafana/grafana
        container_name: grafana
        hostname: grafana
        restart: always
        ports:
            - "3000:3000"
        networks:
            - monitor
            
    redis-exporter:
        image: oliver006/redis_exporter
        container_name: redis_exporter
        hostname: redis_exporter
        restart: always
        ports:
            - "9121:9121"
        networks:
            - monitor
        command:
            - '--redis.addr=redis://10.0.5.79:6379'
            - '--redis.password=Pig_1234'

    node-exporter:
        image: quay.io/prometheus/node-exporter
        container_name: node-exporter
        hostname: node-exporter
        restart: always
        ports:
            - "9100:9100"
        networks:
            - monitor

    mysql-exporter:
        image: prom/mysqld-exporter
        container_name: mysql-exporter
        hostname: mysql-exporter
        restart: always
        ports:
            - "9104:9104"
        networks:
            - monitor
        command:
            - "--mysqld.address=10.0.5.79:3306"
            - "--mysqld.username=pigdigital:Pigdigital_1234"
          
    cadvisor:
        image: google/cadvisor:latest
        container_name: cadvisor
        hostname: cadvisor
        restart: always
        volumes:
            - /:/rootfs:ro
            - /var/run:/var/run:rw
            - /sys:/sys:ro
            - /var/lib/docker/:/var/lib/docker:ro
        ports:
            - "8088:8080"
        networks:
            - monitor
```

`node_down.yml`配置如下：

```yaml
groups:
- name: node_down
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      user: test
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```

## 0x03 启动容器

```shell
# 启动容器
[root@demo-78 prometheus]# docker-compose up -d
```

自行执行命令查看是否启动成功

## 0x04 FAQ

1. 如何使用`prometheus.yml`配置的 "基于文件自动加载新监控任务"

   项目启动后，在根目录会生产一个名叫`reload`的文件夹，只需要在当前文件夹中加入新增的监控配置就能自动注册到`prometheus`服务中,配置书写格式如下：

   ```shell
   [root@demo-78 ~]# cat /prometheus/reload/web.yml 
   - targets:
     - '10.0.5.79:9100'
     labels:
       job: 'pig-web'
       __metrics_path__: '/web/actuator/prometheus'
   ```

   或：

   ```shell
   [root@demo-piggpt-78 ~]# cat /prometheus/reload/mysql.yml 
   - targets: ['10.0.5.78:9104']
   ```

   > 这里只是给个实例，有特别的需求请自行查看资料，这两种写法唯一的区别在于：第一种写法我需要自定义`metrics_path`，否则默认拉取指标的路径（第二种写法）就是：`10.0.5.78:9104/metrics`，对于很多官网的`exporter`，使用第二种方式即可

2. 容器`mysql-export`启动报错：`caller=mysqld_exporter.go:225 level=info msg="Error parsing host config" file=.my.cnf err="no configuration found"`

   由于`mysql-exporter`更新过后，配置写法有改变，截止2024年5月28日，最新的`mysql-exporter:0.15.1`按照当前配置部署能够成功启动，如遇无法启动，请自行查看官方文档

