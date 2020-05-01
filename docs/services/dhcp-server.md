---
title: DHCP Server
description: A quick summary of Dhcp Server
published: true
date: 2019-07-23T02:23:19.249Z
tags: 
---

# DHCP Server

## DHCP Configuration 

Example config: 

```
default-lease-time 600;
max-lease-time 7200;
option domain-name-servers 192.168.10.11, 192.168.10.12;
option domain-name "intranet.example.org";

subnet 192.168.30.0 netmask 255.255.255.0 {
    range 192.168.30.200 192.168.30.249;
    option routers 192.168.30.1;
}
subnet 192.168.9.0 netmask 255.255.255.0 {
    range 192.168.9.200 192.168.9.249;
    option routers 192.168.9.1;
}
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.200 192.168.10.249;
    option routers 192.168.10.1;
}
subnet 192.168.11.0 netmask 255.255.255.0 {
    range 192.168.11.200 192.168.11.249;
    option routers 192.168.11.1;
}
```

## Dynamic DHCP

Updating a dynamic zone: 

https://blog.scottlowe.org/2010/09/07/making-manual-edits-to-dynamic-dns-zones/
