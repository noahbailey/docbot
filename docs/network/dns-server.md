---
title: DNS Server
description: A quick summary of Dns Server
published: true
date: 2019-07-20T19:49:26.374Z
tags: 
---

# DNS Server Configuration 


## Primary DNS Server

Zone files are stored in /var/lib/bind/

Config files are stored in /etc/bind9/named.conf.*

###### `named.conf.local`
```
//  Import the dynamic update key: 
include "/etc/bind/rndc.key";

zone "intranet.example.org" {
        type master;
        file "/var/lib/bind/db.intranet.example.org" ;
        allow-transfer { 192.168.10.12; };
        allow-update {key rndc-key; };
};

zone "192.168.10.in-addr.arpa" {
        type master;
        file "/var/lib/bind/db.192.168.10";
        allow-transfer { 192.168.10.12; };
        allow-update {key rndc-key; };
};
```


###### `named.conf.options`
```
acl "trusted" {
        192.168.0.0/16; 
        localhost; 
}; 

options {
        directory "/var/cache/bind";

        forwarders {
                1.1.1.1;
                1.0.0.1;
        };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };

        allow-recursion { trusted; };
        allow-query-cache { trusted; }; 

        listen-on { 192.168.10.11; 127.0.0.1; };
        allow-transfer { none; };
};
```



## Secondary DNS

###### `named.conf.local`

```
include "/etc/bind/rndc.key";


zone "intranet.example.org" {
        type slave;
        file "/var/lib/bind/db.intranet.example.org" ;
        masters { 192.168.10.11; }; 
};

zone "192.168.10.in-addr.arpa" {
        type slave;
        file "/var/lib/bind/db.192.168.10";
        masters { 192.168.10.11; };
};
```

###### `named.conf.options`

```
acl "trusted" {
        192.168.0.0/16; 
        localhost; 
};

options {
        directory "/var/cache/bind";

        forwarders {
                1.1.1.1;
                1.0.0.1;
        };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };

        allow-recursion { trusted; };
        allow-query-cache { trusted; }; 

        listen-on { 192.168.10.12; 127.0.0.1; };
        allow-transfer { none; };

};
```



# Zone Files

## Forward Lookup

###### `/var/lib/bind/db.intranet.example.org`

```
$TTL    86400
$ORIGIN intranet.example.org.
@  1D  IN  SOA dns1.intranet.example.org. hostmaster.intranet.example.org. (
                              2019032101 ; serial
                              3H ; refresh
                              15 ; retry
                              1w ; expire
                              3h ; nxdomain ttl
                             )
@               IN      NS     dns1.intranet.example.org. 

dns1            IN      A       192.168.10.11
dns2            IN      A       192.168.10.12
server1         IN      A       192.168.10.20
server2         IN      A       192.168.10.22
```


## Reverse Lookup

###### `/var/lib/bind/db.192.168.10`
```
$TTL    86400
$ORIGIN 192.168.10.IN-ADDR.ARPA.
@  1D  IN  SOA dns1.intranet.example.org. hostmaster.intranet.example.org. (
                              2019032101 ; serial
                              3H ; refresh
                              15 ; retry
                              1w ; expire
                              3h ; nxdomain ttl
                             )
@               IN      NS     dns1.intranet.example.org. 

11              IN      PTR     dns1
```