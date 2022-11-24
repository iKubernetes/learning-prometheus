# VictoriaMetrics 示例集群

#### 配置Prometheus以之为远程存储：
```
remote_write:    # 远程写入到远程 VM 存储
  - url: http://vminsert.magedu.com:8480/insert/0/prometheus

remote_read:
  - url: http://vmselect.magedu.com:8481/select/0/prometheus
```

其中的0为多租户模型下的租户ID。


#### 配置Grafana以之为数据源：

添加新数据源，将数据加载路径定义为：http://vmselect.magedu.com:8481/select/0/prometheus

配置完成后，导入Dashboard，即可在Grafana中查看基于该数据源的Dashboard。

