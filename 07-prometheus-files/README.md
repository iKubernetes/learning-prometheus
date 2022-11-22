# Prometheus简明教程

说明：config-examples目录下保存有各类配置示例。

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
chown -R prometheus.prometheus /usr/local/prometheus/data
```

提供Systemd Unitfile，保存于/usr/lib/systemd/system/prometheus.service文件中:
```
[Unit]
Description=Monitoring system and time series database
Documentation=https://prometheus.io/docs/introduction/overview/

[Service]
Restart=always
User=prometheus
EnvironmentFile=/etc/default/prometheus
ExecStart=/usr/local/prometheus/prometheus \
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

### 部署node-exporter

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

提供Systemd Unitfile，保存于/usr/lib/systemd/system/node_exporter.service文件中:
```
[Unit]
Description=node_exporter
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/node_exporter/node_exporter \
  --web.config=web-config.yml \
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

### 部署Grafana

Ubuntu/Debian系统上的部署步骤
```bash
apt-get install -y adduser libfontconfig1
VERSION=9.2.5
curl -LO  ttps://dl.grafana.com/oss/release/grafana_${VERSION}_amd64.deb
dpkg -i grafana_${VERSION}_amd64.deb
```

启动服务：
```bash
systemctl daemon-reload
systemctl start grafana.service
systemctl enable grafana.service
```

验证监听的端口，并测试访问其暴露的指标

```bash
ss -tnlp | grep '3000'
curl localhost:3000/metrics
```

### 部署AlertManager

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
```

提供Systemd Unitfile，保存于/usr/lib/systemd/system/alertmanager.service文件中:
```
[Unit]
Description=node_exporter
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/alertmanager/alertmanager --log.level=debug
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


