# A Prometheus & Grafana docker-compose stack

Here's a quick start to stand-up a [Prometheus](http://prometheus.io/) stack containing Prometheus, [Grafana](https://grafana.com/) and to monitor website uptime.

# Installation & Configuration
At this point you'll have automagically deployed the entire Grafana and Prometheus stack. You can now access the Grafana dashboard at `http://<Host IP Address>:3000` *Username: `admin`, Password: `9uT46ZKE`*. *Note: before the dashboards will work you need to follow the [Datasource Configuration section](#datasource-configuration).*

Here's a list of all the services that are created:

| Service | Port | Description | Notes |
| --- |:---:| --- | --- |
| Prometheus | :9090 | Data Aggregator | |
| Alert Manager | :9093 | Adds Alerting for Prometheus Checks | |
| Grafana | :3000 | UI To Show Prometheus Data | Username: `admin`, Password: `MagedU.C0m`|
| Node Exporter | :9100 | Data Collector for Computer Stats | |
| CA Advisor | :8080 | Collect resource usage of the Docker container | |
| Blackbox Exporter | :9115 | Data Collector for Ping & Uptime | | |

## Post Configuration

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

## Dashboards

Included are two dashboards. You can always find more dashboards on the [Grafana Dashboards Page](https://grafana.com/dashboards?dataSource=prometheus).

### Ping Dashboard

Shows HTTP uptime from websites monitored. See [Ping Configuration](ping-configuration) section.

<img src="images/dashboard-ping.png" alt="Ping Dashboard">

### System Monitoring Dashboard

Shows stats like RAM, CPU, Storage of the current node.

<img src="images/dashboard-system-monitoring.png" alt="System Monitoring Dashboard">

## Alerting

There are 3 basic alerts that have been added to this stack.

| Alert | Time To Fire | Description |
| --- | :---: | --- |
| Site Down | 30 seconds | Fires if a website check is down |
| Service Down | 30 seconds | Fires if a service in this setup is down |
| High Load | 30 seconds | Fires if the CPU load is greater than 50% |

To get alerts sent to you, follow the directions in the [Alert Configuration Section](#alert-configuration).

### Test Alerts
A quick test for your alerts is to simulate high CPU load. Run the utility script `./util/high-load.sh` and about 30 seconds or so later you should notice the incident created in [PagerTree](https://pagertree.com) (assuming you followed the [Alert Configuration Section](#alert-configuration) and you'll also get notifications.

Then `Ctrl+C` to stop this command. The incident should auto resolve in PagerTree.

# Security Considerations
This project is intended to be a quick-start to get up and running with Docker and Prometheus. Security has not been implemented in this project. It is the users responsibility to implement Firewall/IpTables and SSL.

Since this is a template to get started Prometheus and Alerting services are exposing their ports to allow for easy troubleshooting and understanding of how the stack works.

# Troubleshooting
It appears some people have reported no data appearing in Grafana. If this is happening to you be sure to check the time range being queried within Grafana to ensure it is using Today's date with current time.
