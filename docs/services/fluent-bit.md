
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

## Test configuration

Execute the program with `-D` to use dry-run mode: 

    sudo /opt/fluent-bit/bin/fluent-bit -c /etc/fluent-bit/fluent-bit.conf -D

This outputs the results of the config test immediately:

```
Fluent Bit v1.9.8
* Copyright (C) 2015-2022 The Fluent Bit Authors
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

configuration test is successful
```

A non-successful test will output the affected line number and exit with a non-zero code. 


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

[FILTER]
    name record_modifier
    match logs.journald
    record type journald
```

## Input: Nginx events

```ini
[INPUT]
    name tail
    tag logs.nginx
    parser nginx
    path /var/log/nginx/access.log
    db /var/log/nginx/fluent_access.db
    read_from_head true

[FILTER]
    name record_modifier
    match logs.nginx
    record type nginx
```

## GeoIP lookup

Requires the GeoIP database from MaxMind

```ini
[FILTER]
    name geoip2
    match *
    database /var/lib/GeoIP/GeoLite2-City.mmdb
    lookup_key remote
    record geoip_country remote %{country.names.en}
    record geoip_iso remote %{country.iso_code}
    record geoip_city remote %{city.names.en}
    record geoip_latitude remote %{location.latitude}
    record geoip_longitude remote %{location.longitude}
    record geoip_postal_code remote %{postal.code}
    record geoip_region-code remote %{subdivisions.0.iso_code}
    record geoip_region-name remote %{subdivisions.0.names.en}
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

## Output: Forward to another server over HTTP

`/etc/fluent-bit/conf.d/99-forward-http.conf`

Add a `hostname` id to each log before forwarding:

```ini
[FILTER]
    name record_modifier
    match logs.*
    record hostname ${HOSTNAME}

[OUTPUT]
    name http
    match logs.*
    host logs.mycoolserver.com
    uri /flb-log/${HOSTNAME}
    port 443
    tls on
    tls.verify on
    format json
```
