
# Fluentbit

## Install

Key: 

	curl https://packages.fluentbit.io/fluentbit.key | gpg --dearmor > /usr/share/keyrings/fluentbit-keyring.gpg

Configure repo: 

	echo "deb [signed-by=/usr/share/keyrings/fluentbit-keyring.gpg] https://packages.fluentbit.io/debian/bullseye bullseye main" | tee -a /etc/apt/sources.list.d/fluentbit.list

Install package

	apt update && apt install fluent-bit

## Configure

The configuration file at `/etc/fluent-bit/fluent-bit.conf` should just set the basic system config; all inputs and outputs should be contained in granular files. 

```ini
[SERVICE]
	flush 30
	daemon off
	log_level info
	parsers_file parsers.conf
	plugins_file plugins.conf
	http_server  on
	http_listen  127.0.0.1
	http_port    2020
	storage.metrics on
	storage.path /var/log/flb
	storage.sync normal
	storage.backlog.mem_limit 5M

@INCLUDE /etc/fluent-bit/conf.d/*
```

Some directory should also be created: 

	sudo mkdir -p /var/log/flb
	sudo mkdir -p /etc/fluent-bit/conf.d


## Input: Prometheus scraper

Scrapes the node exporter on http://localhost:9100/metrics

`/etc/fluent-bit/conf.d/prom-metrics.conf`

```ini
[INPUT]
	name prometheus_scrape
	host 127.0.0.1
	port 9100
	tag  metrics.prom
	scrape_interval 10s 
```

## Input: Journald

`/etc/fluent-bit/conf.d/journald-logs.conf`

```ini
[INPUT]
	name systemd
	tag logs.journald
	Strip_Underscores on
	db /var/log/journal/fluent.db
	read_from_tail on
	lowercase on
```

## Output: Grafana cloud prometheus remote write

`/etc/fluent-bit/conf.d/99-grafana-cloud-output.conf`

```ini
[OUTPUT]
	name prometheus_remote_write 
	match metrics.*
	host prometheus-prod-10-prod-us-central-0.grafana.net
	uri /api/prom/push
	port 443
	tls on
	tls.verify on
	http_user AAAAA
	http_passwd XXXXXX
	retry_limit false
	add_label host ${HOSTNAME}
```

## Output: Grafana cloud loki service

`/etc/fluent-bit/conf.d/99-grafana-cloud-output.conf`

```ini
[OUTPUT]
	name loki
	match logs.*
	host logs-prod3.grafana.net
	port 443
	tls on
	tls.verify on
	http_user xxxxx
	http_passwd xxxxx
```

## Output: Forward to another server

`/etc/fluent-bit/conf.d/99-forward-output.conf`

```ini
[OUTPUT]
    Name          forward
    Match         logs.*
    Host          logs.mycoolserver.com
    Port          5000
    tls           on
    tls.verify    off
```
