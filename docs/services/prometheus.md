# Prometheus metrics

## Node exporter

    sudo apt install prometheus-node-exporter

### Timers

Edit the timer for apt - change the interval to 2 hrs

`/etc/systemd/system/prometheus-node-exporter-apt.timer`

```ini
[Unit]
Description=Run apt metrics collection every 2 hrs

[Timer]
OnBootSec=0
OnUnitActiveSec=7200

[Install]
WantedBy=timers.target
```

Reload the timer config:

	sudo systemctl daemon-reload

### Exclude systemd stats

These can clutter up the metrics output and cost extra 'series' in grafana's hosted prometheus storage

`/etc/default/prometheus-node-exporter`

```
ARGS="--no-collector.systemd"
```


## Scrapers

Scrapers can be configured to consume the metrics exposed by the exporter. 

Examples:

* [fluent-bit](/services/fluent-bit)
