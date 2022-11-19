# 环境说明

### 静态Target

```yaml
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets:
      - localhost:9090
      - prom.magedu.com:9090
```

测试时，需要确保各主机名能解析到正确的IP地址上；

### 基于文件发现，动态添加Target

```yaml
  # All nodes
  - job_name: 'nodes'
    file_sd_configs:
    - files:                                               
      - targets/nodes-*.yaml  
      refresh_interval: 2m 
```

测试时，需要确保各主机名能解析到正确的IP地址上；
或直接使用部署了node-exporter的节点的IP地址，文件在targets目录下，以“nodes-”为前缀；

### Node Exporter用到的各指标





