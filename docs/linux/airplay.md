# AirPlay server

AirPlay is a proprietary Apple service to allow media streaming through home sound systems. 

A free-software server also exists to implement this using a linux system. 


### Install

First, install the package:

    sudo apt install shairport-sync

### Configure

Some lines in the config file must be changed: 

`/etc/shairport-sync.conf`

```
general = 
{
    name = "Bathroom PC";
}
...
alsa =
{
    output_device = "hw:PCH";
}
```

### Networking

AirPlay relies on 'Bonjour' which is mDNS (multicast DNS). Make sure the `avahi-daemon` service is running: 

    sudo systemctl start avahi-daemon.service 

Also make sure the ports are allowed in the firewall: 

    sudo iptables -A INPUT -d 224.0.0.251/32 -p udp -m udp --dport 5353 -j ACCEPT

