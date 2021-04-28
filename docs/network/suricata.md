# Suricata Network IDS

## Install Suricata

Add the apt repository: 

	sudo add-apt-repository ppa:oisf/suricata-stable
	sudo apt update

Install the software: 

    sudo apt install suricata

## Configure Suricata

After installing, the only tweaks needed to get up and running are to change the interfaces and set up the network ranges.  

```yaml
vars: 
  address-groups: 
    HOME_NET: "[10.98.76.0/24]"
    EXTERNAL_NET: "!$HOME_NET"
```

And, further down,

```yaml
af-packet: 
  - interface: eth0
    threads: auto 
  - interface: eth1
    threads: auto
```

The only other tweak to make to the default config is to disable the built in rules and let `suricata-update` manage rule updates. 

```yaml
default-rule-path: /var/lib/suricata/rules
rule-files: 
  - suricata.rules
```

The service should start up and run at this point. 

    sudo systemctl enable --now suricata

## Rulestes

Update the rulesets

```
sudo suricata-update update-sources
sudo suricata-update list-sources

sudo suricata-update enable-source oisf/trafficid
sudo suricata-update enable-source et/open
sudo suricata-update enable-source sslbl/ssl-fp-blacklist
```

After rulesets are enabled, update the rules by live reloading the service: 

	sudo suricata-update && kill -USR2 $(pidof suricata)

## Automated rule updates

Create a file in `/etc/cron.d/suricata` with this content: 

```cron
# Daily suricata update
00 0,6,12,18 * * *  root  (suricata-update && kill -USR2 `pidof suricata`)
```
