# IPTables NAT 

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
COMMIT
```

## Apply the rules

    sudo iptables-restore rules.nat

## View applied rules

```
$ sudo iptables -L -v -t nat
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       tcp  --  any    any     anywhere             anywhere             tcp dpt:3000 to:192.168.122.200:3000
```
