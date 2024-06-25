# 基于Consul的服务发现示例

### 基于Consul的Job

```yaml
  # All nodes
  - job_name: 'nodes'
    consul_sd_configs:
    - server: "consul:8500"
      tags:
      - "nodes"
      refresh_interval: 2m 
```

基于Consul完成服务发现，但相关的节点的信息以静态形式进行添加，以简化测试过程；相关的节点信息保存于consul_configs目录下json格式的配置文件中

### 创建Job

下面的示例，创建了一个job，它从consul中发现标签为“grafana”的服务，并将其自动添加为target。

```yaml
  # grafana
  - job_name: 'grafana'
    consul_sd_configs:
    - server: "consul:8500"
      tags:
      - "grafana"
      refresh_interval: 2m 
```

下面的示例，创建了一个job，它从consul中发现标签为“consul-exporter”的服务，并将其自动添加为target。

```yaml
  # consul
  - job_name: 'consul'
    consul_sd_configs:
    - server: "consul:8500"
      tags:
      - "consul-exporter"
      refresh_interval: 2m 
```

### 动态注册服务示例

示例1：下面的配置能够将grafana注册到consul上，相关的内容保存于grafana.json文件中。

```json
{
    "id": "grafana01",
    "name": "grafana01",
    "address": "grafana",
    "port": 3000,
    "tags": ["grafana"],
    "checks": [{
      "http": "http://grafana:3000/metrics",
      "interval": "5s"
    }]
}
```

示例2：将consul-exporter注册到consul上，从而让prometheus可以监控consul。相关的内容保存于consul-exporter.json文件中。

```json
{
      "id": "consul-exporter",
      "name": "consul-exporter",
      "address": "172.29.2.12",
      "port": 9107,
      "tags": ["consul-exporter"],
      "checks": [{
        "http": "http://172.29.2.12:9107/metrics",
        "interval": "5s"
      }]
}
```

### 注册命令

```bash
curl -XPUT --data @grafana.json http://consul:8500/v1/agent/service/register
curl -XPUT --data @consul-exporter.json http://consul:8500/v1/agent/service/register
```



