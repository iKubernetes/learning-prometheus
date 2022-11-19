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

### Grafana Job

为发现的grafana定义的一个专用Job示例，发现时使用的过滤标签为“grafana”。

```yaml
  # grafana
  - job_name: 'grafana'
    consul_sd_configs:
    - server: "consul:8500"
      tags:
      - "grafana"
      refresh_interval: 2m 
```

### 动态注册服务示例

要注册的grafana配置示例

```json
{
  "service": {
    "id": "grafana",
    "name": "grafana01",
    "address": "grafana",
    "port": 9100,
    "tags": ["grafana"],
    "checks": [{
      "http": "http://grafana:3000/metrics",
      "interval": "5s"
    }]
  }
}
```

### 注册命令

```bash
curl -XPUT --data @grafana.json http://consul:8500/v1/agent/service/register
```



