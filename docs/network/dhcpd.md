# DHCP Server

## Install

    sudo apt install -y isc-dhcp-server

The service will go into failed state immediately - That is normal. 

## Configure

Edit `/etc/dhcp/dhcpd.conf`

```
option domain-name "example.com";
option domain-name-servers 10.98.76.1, 1.1.1.1;

default-lease-time 7200;
max-lease-time 86400;
deny declines;
deny duplicates;
one-lease-per-client true;

ddns-update-style none;

subnet 10.98.76.0 netmask 255.255.255.0 {
  authoritative;
  range 10.98.76.50 10.98.76.250;
  option routers 10.98.76.1;
}
```

## Start the service

Test the server config: 

    sudo dhcpd -t

```
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
```

Test the lease file: 

    sudo dhcpd -T

```
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
Wrote 79 leases to leases file.
Lease file test successful, removing temp lease file: /var/lib/dhcp/dhcpd.leases.1649716327
```

Start the service: 

    sudo systemctl enable isc-dhcp-server
    sudo systemctl start isc-dhcp-server
