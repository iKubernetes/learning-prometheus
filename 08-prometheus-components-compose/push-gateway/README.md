# Push Gateway



在Prometheus上创建Job。

```yaml
scrape_configs:
- job_name: pushgateway
  honor_labels: false
  static_configs:
  - targets: ['pushgw.magedu.com:9091']
    labels:
      pushgateway_instance: metricfire
```



手动提交指标进行测试。

```bash
cat <<EOF | curl --data-binary @- http://localhost:9091/metrics/job/metricfire/instance/article
# TYPE my_test_metric gauge
my_test_metric 77
# TYPE awesomeness_total counter
# HELP awesomeness_total How awesome is this article.
awesomeness_total 
987654321
EOF
```



### Push Gateway

Push Gateway的常用选项。

- *— web.listen-address=:9091*, IP (optional) and port pair on which to listen for requests;
- *— web.telemetry-path=/metrics*, the Path under which the metrics (both user-sent and internal ones) of the Pushgateway will be exposed;
- *— web.external-url=*, URL on which this Pushgateway is externally available. Useful if you expose it via some domain name;
- *— web.route-prefix=*, if specified then uses this as a prefix for all of the routes. Defaults to *— web. external-url*’s prefix;
- *— web.enable-lifecycle*, if specified then lets you shutdown the Pushgateway via the [API](https://github.com/prometheus/pushgateway#management-api);
- *— web.enable-admin-api*, if specified then enables the Admin API. It lets you perform certain destructive actions. More on that in the following sections;
- *— persistence.file=*, if specified then Pushgateway writes its state to this file every *— persistence.interval* time period;
- *— persistence.interval=5m*, how often the state should be written to the previously specified file;
- *— push.disable-consistency-check*, if specified then the metrics are not checked for correctness at ingestion time. Should not be specified in the absolute majority of cases;
- *— log.level=info*, one of *debug*, *info*, *warn*, *error*. Only prints messages with levels higher than that;
- *— log.format=logfmt*, possible values: *logfmt*, *json*. Specify *json* if you want structured logs that could be used with, for example, [Elasticsearch](https://www.elastic.co/).
