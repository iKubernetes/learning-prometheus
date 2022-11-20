# 多种Exporter监控示例

- consul-exporter
- mysqld-exporter
- nginx-exporter
- tomcat

### 各关键服务端口

容器环境本地网络：172.31.0.0/16

- prometheus: 9090/tcp
- consul： 8500/tcp
- tomcat: 8080/tcp
- grafana: 3000/tcp
- alertmanager: 9093/tcp

## mysqld-exporter使用说明

需要在mysqld上运行如下命令，为mysqld-exporter创建具有采集监控数据权限的用户账号

```
  mysql> CREATE USER 'exporter'@'mysqld-exporter' IDENTIFIED BY 'exporter';
  mysql> GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'mysqld-exporter';
  mysql> GRANT SELECT ON performance_schema.* TO 'exporter'@'mysqld-exporter';
```

本示例中，mysqld和mysqld-exporter均运行于172.31.0.0/16网络中，因此，也可按如下方式完成用户创建及授权

```
  mysql> CREATE USER 'exporter'@'172.31.%.%' IDENTIFIED BY 'exporter';
  mysql> GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'172.31.%.%';
  mysql> GRANT SELECT ON performance_schema.* TO 'exporter'@'172.31.%.%';
```

注意：在docker-compose.yml文件中，mysqld-exporter接入mysqld的用户密和密码默认为“exporter”，因此，或需修改，请同时修改该文件中的配置；

#### mysqld监控可用的一些示例记录规则

```
groups:
- name: mysqld_rules
  rules:

  # Record slave lag seconds for pre-computed timeseries that takes
  # `mysql_slave_status_sql_delay` into account
  - record: instance:mysql_slave_lag_seconds
    expr: mysql_slave_status_seconds_behind_master - mysql_slave_status_sql_delay

  # Record slave lag via heartbeat method
  - record: instance:mysql_heartbeat_lag_seconds
    expr: mysql_heartbeat_now_timestamp_seconds - mysql_heartbeat_stored_timestamp_seconds

  - record: job:mysql_transactions:rate5m
    expr: sum without (command) (rate(mysql_global_status_commands_total{command=~"(commit|rollback)"}[5m]))
```

#### mysqld监控可用的示例告警规则

```
groups:
- name: MySQLAlerts
  rules:
  - alert: MySQLDown
    expr: mysql_up != 1
    for: 5m
    labels:
      severity: critical
    annotations:
      description: 'MySQL {{$labels.job}} on {{$labels.instance}} is not up.'
      summary: MySQL not up.  
  - alert: MySQLReplicationNotRunning
    expr: mysql_slave_status_slave_io_running == 0 or mysql_slave_status_slave_sql_running
      == 0
    for: 2m
    labels:
      severity: critical
    annotations:
      description: "Replication on {{$labels.instance}} (IO or SQL) has been down for more than 2 minutes."
      summary: Replication is not running.
  - alert: MySQLReplicationLag
    expr: (instance:mysql_slave_lag_seconds > 30) and on(instance) (predict_linear(instance:mysql_slave_lag_seconds[5m],
      60 * 2) > 0)
    for: 1m
    labels:
      severity: critical
    annotations:
      description: "Replication on {{$labels.instance}} has fallen behind and is not recovering."
      summary: MySQL slave replication is lagging.
  - alert: MySQLHeartbeatLag
    expr: (instance:mysql_heartbeat_lag_seconds > 30) and on(instance) (predict_linear(instance:mysql_heartbeat_lag_seconds[5m],
      60 * 2) > 0)
    for: 1m
    labels:
      severity: critical
    annotations:
      description: "The heartbeat is lagging on {{$labels.instance}} and is not recovering."
      summary: MySQL heartbeat is lagging.
  - alert: MySQLInnoDBLogWaits
    expr: rate(mysql_global_status_innodb_log_waits[15m]) > 10
    labels:
      severity: warning
    annotations:
      description: The innodb logs are waiting for disk at a rate of {{$value}} /
        second
      summary: MySQL innodb log writes stalling.
```

## Consul Exporter有用的查询示例

**Are my services healthy?**

```
min(consul_catalog_service_node_healthy) by (service_name)
```

Values of 1 mean that all nodes for the service are passing. Values of 0 mean at least one node for the service is not passing.

**处于宕机状态的服务节点**

```
sum by (node, service_name)(consul_catalog_service_node_healthy == 0)
```

**获取处于critical状态的服务**

```
consul_health_service_status{status="critical"} == 1
```

You can query for the following health check states: "maintenance", "critical", "warning" or "passing"

## BlackBox Exporter

### Ping Configuration

If you would like to add or change the Ping targets should be monitored you'll want to edit the `targets` section in [prometheus/prometheus.yml](prometheus/prometheus.yml)

```yml
...

- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
      - https://magedu.com # edit here
      - https://google.com # edit here

...
```

If you made changes to the Prometheus config you'll want to reload the configuration using the following command:

```curl
curl -X POST http://<Host IP Address>:9090/-/reload
```

### Alert Configuration
The [PagerTree](https://pagertree.com) configuration requires to create a Prometheus Integration. Follow steps 1-6 [here](https://pagertree.com/knowledge-base/integration-prometheus/#in-pagertree) then replace `https://ngrok.io` in [/alertmanager/config.yml](/alertmanager/config.yml) with your copied webhook.

```yml
...
receivers:
    - name: 'email'
      email_configs:
...
```

If you made changes to the AlertManager config you'll want to reload the configuration using the following command:

```curl
curl -X POST http://<Host IP Address>:9093/-/reload
```

## Alerting

There are 3 basic alerts that have been added to this stack.

| Alert | Time To Fire | Description |
| --- | :---: | --- |
| Site Down | 30 seconds | Fires if a website check is down |
| Service Down | 30 seconds | Fires if a service in this setup is down |
| High Load | 30 seconds | Fires if the CPU load is greater than 50% |

To get alerts sent to you, follow the directions in the [Alert Configuration Section](#alert-configuration).
