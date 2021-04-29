# IPTables NAT Port Forwarding

## Kernel Params

Enable packet forwarding: 

    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf

Reload sysctls: 

    sudo sysctl --system

## Install startup scripts

Install the package: 

    sudo apt-get install -y iptables-persistent

This will create a set of startup services to load the contents of `/etc/iptables/rules.*` on boot. 

## Port forwarding

`rules.nat`

```sh
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
# NAT rules for load-balancer
-A PREROUTING -p tcp -m tcp --dport 3000 -j DNAT --to-destination 192.168.122.200:3000
# Masquerading -- Needed for crossing network boundaries 
-A POSTROUTING -o enp1s0 -j MASQUERADE
COMMIT
```

## Apply the rules

    sudo iptables-restore rules.nat

## View applied rules

```
noah@nat:~$ sudo iptables -L -v -t nat
Chain PREROUTING (policy ACCEPT 22 packets, 2812 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    1    60 DNAT       tcp  --  any    any     anywhere             anywhere             tcp dpt:3000 to:192.168.122.200:3000

Chain INPUT (policy ACCEPT 10 packets, 792 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 8 packets, 662 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 3 packets, 246 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    6   476 MASQUERADE  all  --  any    enp1s0  anywhere             anywhere            

```
