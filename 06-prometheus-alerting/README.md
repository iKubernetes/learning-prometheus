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


## Alerting

### Rules Configuration

If you would like to add or change the Ping targets should be monitored you'll want to edit the `targets` section in [prometheus/prometheus.yml](prometheus/prometheus.yml)

```yml
...

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager:9093
    scheme: http

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - rules/alert-rules-*.yml
  - rules/record-rules-*.yml
  # - "first_rules.yml"
  # - "second_rules.yml"

...
```

If you made changes to the Prometheus config you'll want to reload the configuration using the following command:

```curl
curl -X POST http://<Host IP Address>:9090/-/reload
```

### Alert Configuration

告警消息默认发给team-devops-dingtalk，其它的按需进行路由。

```yml
route:
    # The labels by which incoming alerts are grouped together. For example,
    # multiple alerts coming in for job=mysqld-exporter and alertname=mysql_down would
    # be batched into a single group.
    group_by: ['job', 'alertname'] 

    # When a new group of alerts is created by an incoming alert, wait at
    # least 'group_wait' to send the initial notification.
    # This way ensures that you get multiple alerts for the same group that start
    # firing shortly after another are batched together on the first notification.
    group_wait: 10s

    # When the first notification was sent, wait 'group_interval' to send a batch
    # of new alerts that started firing for that group.
    group_interval: 10s

    # If an alert has successfully been sent, wait 'repeat_interval' to resend them.
    repeat_interval: 30m

    receiver: team-devops-dingtalk
    #receiver: wechat
    #receiver: webhook

    # The child route trees.
    routes:
    # This routes performs a regular expression match on alert labels to
    # catch alerts that are related to a list of jobs.
    - matchers:
        - job =~ "mysql|tomcat"
      receiver: team-devops-email

      # The service has a sub-route for critical alerts, any alerts
      # that do not match, i.e. severity != critical, fall-back to the
      # parent node and are sent to 'team-devops-email'
      routes:
        - matchers:
            - severity = "critical"
          receiver: team-devops-dingtalk

    - matchers:
        - job = "nginx-exporter"
      receiver: team-devops-email

      routes:
        - matchers:
            - severity = "critical"
          receiver: team-devops-wechat

templates:
  - '/etc/alertmanager/email_template.tmpl'

 # 定义接收者
receivers:
- name: 'team-devops-email'
  email_configs:
    - to: 'mage@magedu.com'
      headers:
        subject: "{{ .Status | toUpper }} {{ .CommonLabels.env }}:{{ .CommonLabels.cluster }} {{ .CommonLabels.alertname }}"
      html: '{{ template "email.to.html" . }}'
      send_resolved: true 

- name: 'team-devops-wechat'
  wechat_configs:
    - corp_id: ww4c893...
      to_user: '@all'
      agent_id: 1000008
      api_secret: WTepmmaqxbBOeTQ...
      send_resolved: true

- name: 'team-devops-dingtalk'
  webhook_configs:
  - url: http://prometheus-webhook-dingtalk:8060/dingtalk/webhook1/send             
    send_resolved: true

inhibit_rules: 
  - source_match: 
     severity: 'critical' 
    target_match: 
     severity: 'warning' 
    equal: ['alertname', 'job']  
```

If you made changes to the AlertManager config you'll want to reload the configuration using the following command:

```curl
curl -X POST http://<Host IP Address>:9093/-/reload
```

### 钉钉告警测试说明

dingtalk目录下提供了三个配置文件，它们都可以复制为config.yml，以作为默认被加载的配置文件进行告警测试。

- config-no-template.yml： 没有使用消息模板的配置文件；
- config-use-default-template.yml：使用了内置消息模板的配置文件；
- config-use-customed-template.yml：使用了自定义消息模板的配置文件；




