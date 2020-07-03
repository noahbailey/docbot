---
title: MikroTik OpenVPN Server
description: 
published: true
date: 2019-07-20T19:49:43.395Z
tags: 
---

## MikroTik OpenVPN Server

### Create CA certs

Generate certificate templates, 2048 key size and valid for 10 years. 

    /certificate add name=ca-template common-name=example.org days-valid=3650 key-size=2048 key-usage=crl-sign,key-cert-sign
    /certificate add name=server-template common-name=*.example.org days-valid=3650 key-size=2048 key-usage=digital-signature,key-encipherment,tls-server
    /certificate add name=client-template common-name=client.example.org days-valid=3650 key-size=2048 key-usage=tls-client

Generate and sign certificates based on the tamplates

    /certificate sign ca-template name=ca-certificate
    /certificate sign server-template name=server-certificate ca=ca-certificate
    /certificate sign client-template name=client-certificate ca=ca-certificate

Export PKI certs to be available for client connections: 

    /certificate export-certificate ca-certificate export-passphrase=""
    /certificate export-certificate client-certificate export-passphrase="passssssssssssssssssword"

### Create VPN Interfaces

Create the DHCP pool for the ovpn interface: 

    /ip pool add name=ovpn-pool ranges=192.168.255.10-192.168.255.20

Create a PPP network policy: 

    /ppp profile add change-tcp-mss=default comment="" local-address=192.168.255.1 name="openvpn-profile" only-one=default remote-address=ovpn-pool use-compression=default use-encryption=required

Create a PPP user and secret (password for remote connection)

    /ppp secret add caller-id="" disabled=no limit-bytes-in=0 limit-bytes-out=0 name=noah password="passsssssword" routes="" service=any
    
>Todo: Experiment with pushing routes over this policy. Example, 192.168.10.0/16

Assign the certificates and crypto map to the ovpn interface: 

    /interface ovpn-server server set auth=sha1 the-certificate-generated-above.pem cipher=aes128,aes192,aes256 default-profile=openvpn-profile enabled=yes keepalive-timeout=disabled max-mtu=1500 mode=ip netmask=24 port=1194 require-client-certificate=no


    /interface ovpn-server server set default-profile=openvpn-profile certificate=server-certificate require-client-certificate=yes auth=sha1 cipher=aes128,aes192,aes256 enabled=yes

