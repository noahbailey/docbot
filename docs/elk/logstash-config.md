---
title: Logstash Config
description: Configuration Reference for Logstash
published: true
date: 2019-07-20T19:49:38.298Z
tags: 
---

# Logstash Configuration 

Logstash configurations are in three main phases: input, filter, and output. 

All of the configurations are located in the directory, `/etc/logstash/conf.d/` on the ELK server. 

## Ingest Pipeline

This configuration two inputs, and more may be added in the future. Thanks to the modular nature of logstash this will be no issue to maintain. 

### Filebeat Input

`10-beats-input.conf`

```
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
```

### Syslog Input

`11-syslog-input.conf`

```
input {
  syslog {
    port => 5514
  }
}
```

## Processing

In this pipeline, logs are sorted and modified according to their type. 

Syslog entries are modified to contain only relevant fields, host headers are discarded, and a timestamp is added. 

Web logs (delivered via the beats input) are parsed as well, with the geoip fields added. 

`20-transform.conf`

```
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
  grok {
    match => [ "message" , "%{COMBINEDAPACHELOG}+%{GREEDYDATA:extra_fields}"]
    overwrite => [ "message" ]
  }
  mutate {
    remove_field => [ "[host][name]" ]
    remove_field => [ "[host][id]" ]
    remove_field => [ "[host][architecture]" ]
    remove_field => [ "[host][os][platform]" ]
    remove_field => [ "[host][os][version]" ]
    remove_field => [ "[host][os][family]" ]
    remove_field => [ "[host][ip]" ]
    remove_field => [ "[host][mac]" ]
    remove_field => [ "[host][os]" ]
    remove_field => [ "[host]" ]
  }
  mutate {
    convert => ["response", "integer"]
    convert => ["bytes", "integer"]
    convert => ["responsetime", "float"]
  }
  geoip {
    source => "clientip"
    target => "geoip"
    add_tag => [ "nginx-geoip" ]
  }
  date {
    match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
    remove_field => [ "timestamp" ]
  }
  useragent {
    source => "agent"
  }
}
```


## Output

Processed logs are sent to Elasticsearch for storage and indexing. All logs are added to the `logstash` index pattern for simplicity. 

```
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```


