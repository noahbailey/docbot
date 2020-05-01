---
title: Suricata from Source
description: Network layer IDS/IPS server
published: true
date: 2019-11-28T03:52:01.661Z
tags: 
---

# Suricata	

https://suricata.org 

https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Debian_Installation

## Installation 

Install the build prerequisites: 

```
sudo apt install -y libpcre3 libpcre3-dbg libpcre3-dev \
	build-essential autoconf automake libtool libpcap-dev libnet1-dev \
	libyaml-0-2 libyaml-dev zlib1g zlib1g-dev libmagic-dev libcap-ng-dev \
	libjansson-dev pkg-config libnspr4-dev libnss3-dev liblz4-dev
```

### Additional prereqs: 

- For IPS functionality: 
```
	apt-get -y install libnetfilter-queue-dev
```

- For newer versions that use Rust: 
```
	sudo apt install  rustc cargo
```

- Additional Python modules: 
```
	sudo apt install python3-yaml python3-distutils
```

Download sources: 

```
wget http://www.openinfosecfoundation.org/download/suricata-5.0.0.tar.gz
tar -xvzf suricata-5.0.0.tar.gz
cd suricata-5.0.0
```

### Compile with IPS

```
./configure --enable-nfqueue --prefix=/usr --sysconfdir=/etc --localstatedir=/var
```

### Compile Without IPS

```
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var
```

### Install to system 

Full install: 

```
make && make install-full
```



## Start and Run

Update the rulesets

```
sudo suricata-update

sudo suricata-update update-sources
sudo suricata-update list-sources

sudo suricata-update enable-source oisf/trafficid
sudo suricata-update enable-source et/open
sudo suricata-update enable-source sslbl/ssl-fp-blacklist
```

Start the service: 

```
sudo systemctl start suricata.service
```



### Oinkmaster

```
sudo apt install oinkmaster
```

add the ET rules URL: 

```
url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
```

And run the rule updater: 

```
sudo oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```



TODO: Create updater scripts, config log rotation etc. 

TODO: Enable drop mode
Line 425: 

```yaml
- drop: 
    enabled: no --> yes
```

