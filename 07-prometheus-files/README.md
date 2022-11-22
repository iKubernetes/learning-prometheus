# Prometheus简明教程

**说明**
- 本示例中的系统环境为Ubuntu Server 20.04.5 X86_64
- config-examples目录下保存有各类配置示例

## Server端组件

以下组件可以部署在不同的节点之上，彼此间能通过网络互通即可。

### 部署Prometheus Server

下载程序包，以2.40.2版为例：
```bash
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.40.2/prometheus-2.40.2.linux-amd64.tar.gz
```

展开程序包：
```bash
tar xf prometheus-2.40.2.linux-amd64.tar.gz -C /usr/local/
ln -sv /usr/local/prometheus-2.40.2.linux-amd64 /usr/local/prometheus
```

创建用户，并设定目录权限：
```bash
useradd -r prometheus
mkdir /usr/local/prometheus/data
chown -R prometheus.prometheus /usr/local/prometheus/data
```

创建Systemd Unitfile，保存于/usr/lib/systemd/system/prometheus.service文件中:
```
[Unit]
Description=Monitoring system and time series database
Documentation=https://prometheus.io/docs/introduction/overview/

[Service]
Restart=always
User=prometheus
EnvironmentFile=-/etc/default/prometheus
ExecStart=/usr/local/prometheus/prometheus \
            --config.file=/usr/local/prometheus/prometheus.yml \
            --storage.tsdb.path=/usr/local/prometheus/data \
            --web.console.libraries=/usr/share/prometheus/console_libraries \
            --web.enable-lifecycle \
            $ARGS
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no
LimitNOFILE=8192

[Install]
WantedBy=multi-user.target
```

如有必要，可创建环境配置文件/etc/default/prometheus，通过变量ARGS为prometheus指定启动参数


启动服务：
```bash
systemctl daemon-reload
systemctl start prometheus.service
systemctl enable prometheus.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '9090'
curl localhost:9090/metrics
```

修改配置后的重载命令：
```bash
curl -XPOST http://localhost:9090/-/reload
```

随后即可访问Prometheus的Web UI，其使用的URL如下，其中的<HOST_IP>要替换为节点的实际地址:
http://<HOST_IP>:9090/

### 部署Consul

组件功能：用于为Prometheus提供基于Consul进行服务发现的测试环境。

下载Consul，以1.14.1版本为例：
```bash
curl -LO https://releases.hashicorp.com/consul/1.14.1/consul_1.14.1_linux_amd64.zip
```

展开程序包：

```bash
mkdir -p /usr/local/consul/config/data
unzip consul_1.14.1_linux_amd64.zip -d /usr/local/consul
```

创建用户，或prometheus用户已经存在，可略过该步骤：
```bash
useradd -r consul
chown consul.consul /usr/local/consul/{data,config}
```

创建Systemd Unitfile，保存于/usr/lib/systemd/system/consul.service文件中:
```
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target

[Service]
EnvironmentFile=-/etc/consul.d/consul.env
User=consul
Group=consul
ExecStart=/usr/bin/consul agent -dev -bootstrap \
            -config-dir /usr/local/consul/config \
            -data-dir /usr/local/consul/data \
            -ui \
            -log-level INFO \
            -bind 127.0.0.1 \
            -client 0.0.0.0
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGTERM
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

启动服务：
```bash
systemctl daemon-reload
systemctl start consul.service
systemctl enable consul.service
```

验证监听的端口

```bash
ss -tnlp | grep '8500'
```

随后即可访问Consul的Web UI，其使用的URL如下，其中的<HOST_IP>要替换为节点的实际地址:
http://<HOST_IP>:8500/


### 部署Grafana

组件功能：用于为Proemtheus提供完善的可视化界面。

Ubuntu/Debian系统上的部署步骤
```bash
apt-get install -y adduser libfontconfig1
VERSION=9.2.5
curl -LO https://dl.grafana.com/oss/release/grafana_${VERSION}_amd64.deb
dpkg -i grafana_${VERSION}_amd64.deb
```

启动服务：
```bash
systemctl daemon-reload
systemctl start grafana-server.service
systemctl enable grafana-server.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '3000'
curl localhost:3000/metrics
```

随后即可访问Grafana Server的Web UI，其使用的URL如下，其中的<HOST_IP>要替换为节点的实际地址:
http://<HOST_IP>:3000/

### 部署AlertManager

组件功能：用于为Proemtheus提供发送告警的通知的系统组件；

下载程序包，以0.24.0版为例：
```bash
curl -LO https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
```

展开程序包：
```bash
tar xf alertmanager-0.24.0.linux-amd64.tar.gz -C /usr/local/
ln -sv /usr/local/alertmanager-0.24.0.linux-amd64 /usr/local/alertmanager
```

创建用户，或prometheus用户已经存在，可略过该步骤：
```bash
useradd -r prometheus
mkdir /usr/local/alertmanager/data
chown -R prometheus.prometheus /usr/local/alertmanager/data
```

创建Systemd Unitfile，保存于/usr/lib/systemd/system/alertmanager.service文件中:
```
[Unit]
Description=alertmanager
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/alertmanager/alertmanager \
            --config.file="/usr/local/alertmanager/alertmanager.yml" \
            --storage.path="/usr/local/alertmanager/data/" \
            --data.retention=120h \
            --log.level=info
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
Restart=always

[Install]
WantedBy=multi-user.target
```

启动服务：
```bash
systemctl daemon-reload
systemctl start alertmanager.service
systemctl enable alertmanager.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '9093'
curl localhost:9093/metrics
```

修改配置后的重载命令：
```bash
curl -XPOST http://localhost:9093/-/reload
```

随后即可访问AlertManager的Web UI，其使用的URL如下，其中的<HOST_IP>要替换为节点的实际地址:
http://<HOST_IP>:9093/


## Exporters

### 部署node-exporter

提示：每个主机节点上均应该部署node-exporter；

下载程序包，以1.4.0版本为例：
```bash
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
```

展开程序包：

```bash
tar xf node_exporter-1.4.0.linux-amd64.tar.gz -C /usr/local/
ln -sv /usr/local/node_exporter-1.4.0.linux-amd64 /usr/local/node_exporter
```

创建用户，或prometheus用户已经存在，可略过该步骤：
```bash
useradd -r prometheus
```

创建Systemd Unitfile，保存于/usr/lib/systemd/system/node_exporter.service文件中:
```
[Unit]
Description=node_exporter
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/node_exporter/node_exporter \
  --collector.ntp \
  --collector.mountstats \
  --collector.systemd \
  --collector.ethtool \
  --collector.tcpstat
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
Restart=always

[Install]
WantedBy=multi-user.target
```

启动服务：
```bash
systemctl daemon-reload
systemctl start node_exporter.service
systemctl enable node_exporter.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '9100'
curl localhost:9100/metrics
```

### 部署Consul Exporter

提示：仅需要为每个Consul实例部署consul-exporter，它负责将Consul的状态信息转为Prometheus兼容的指标格式并予以暴露。

下载程序包，以0.8.0版本为例：
```bash
curl -LO https://github.com/prometheus/consul_exporter/releases/download/v0.8.0/consul_exporter-0.8.0.linux-amd64.tar.gz
```

展开程序包：

```bash
tar xf consul_exporter-0.8.0.linux-amd64.tar.gz -C /usr/local/
ln -sv /usr/local/consul_exporter-0.8.0.linux-amd64 /usr/local/consul_exporter
```

创建用户，或prometheus用户已经存在，可略过该步骤：
```bash
useradd -r consul
```

创建Systemd Unitfile，保存于/usr/lib/systemd/system/consul_exporter.service文件中:
```
[Unit]
Description=consul_exporter
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=consul
EnvironmentFile=-/etc/default/consul_exporter
# 具体使用时，若consul_exporter与consul server不在同一主机时，consul server要指向实际的地址；
ExecStart=/usr/local/consul_exporter/consul_exporter \
            --consul.server="http://localhost:8500" \
            --web.listen-address=":9107" \
            --web.telemetry-path="/metrics" \
            --log.level=info \
            $ARGS
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
Restart=always

[Install]
WantedBy=multi-user.target
```

启动服务：
```bash
systemctl daemon-reload
systemctl start consul_exporter.service
systemctl enable consul_exporter.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '9107'
curl localhost:9107/metrics
```

### 部署MySQL Exporter

提示：仅需要为每个MySQL Server实例部署mysql-exporter，它负责将MySQL Server的状态信息转为Prometheus兼容的指标格式并予以暴露。

下载程序包，以0.14.0版本为例：
```bash
curl -LO https://github.com/prometheus/mysqld_exporter/releases/download/v0.14.0/mysqld_exporter-0.14.0.linux-amd64.tar.gz
```

展开程序包：

```bash
tar xf mysqld_exporter-0.14.0.linux-amd64.tar.gz -C /usr/local/
ln -sv /usr/local/mysqld_exporter-0.14.0.linux-amd64 /usr/local/mysqld_exporter
```

创建用户，或prometheus用户已经存在，可略过该步骤：
```bash
useradd -r mysql
```

创建Systemd Unitfile，保存于/usr/lib/systemd/system/mysqld_exporter.service文件中:
```
[Unit]
Description=consul_exporter
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=consul
EnvironmentFile=-/etc/default/mysqld_exporter
# 具体使用时，若mysql_exporter与mysql server不在同一主机时，mysql server要指向实际的地址；
# mysql_exporter连接mysql server使用的用户名和密码均为exporter，该用户要获得正确的授权；
Environment='DATA_SOURCE_NAME=exporter:exporter@(localhost:3306)'
ExecStart=/usr/local/mysqld_exporter/mysqld_exporter \
            --web.listen-address=":9104" \
            --web.telemetry-path="/metrics" \
            --collect.info_schema.innodb_tablespaces \
            --collect.info_schema.innodb_metrics \
            --collect.global_status \
            --collect.global_variables \
            --collect.slave_status \
            --collect.engine_innodb_status \
            $ARGS
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
Restart=always

[Install]
WantedBy=multi-user.target
```

在mysqld server上添加用户，并授权其能够加载mysql的信息并转换为指标输出。需要注意的是用户账号授权时使用的主机范围。
```
mysql> CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'exporter';
mysql> GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'localhost';
mysql> GRANT SELECT ON performance_schema.* TO 'exporter'@'localhost';
mysql> FLUSH PRIVILEGES;
```

启动服务：
```bash
systemctl daemon-reload
systemctl start mysqld_exporter.service
systemctl enable mysqld_exporter.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '9104'
curl localhost:9104/metrics
```





