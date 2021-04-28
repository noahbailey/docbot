# DHCP Server

## Install

    sudo apt install -y isc-dhcp-server

The service will go into failed state immediately - That is normal. 

## Configure

Edit `/etc/dhcp/dhcpd.conf`

```
option domain-name "example.com";
option domain-name-servers 10.98.76.1, 1.1.1.1;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style none;

subnet 10.98.76.0 netmask 255.255.255.0 {
  authoritative;
  range 10.98.76.50 10.98.76.250;
  option routers 10.98.76.1;
}
```

## Start the service

    sudo systemctl enable isc-dhcp-server
    sudo systemctl start isc-dhcp-server
