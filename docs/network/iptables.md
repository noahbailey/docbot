# IPTables Firewall

## Kernel Params

Enable packet forwarding: 

    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf

Reload sysctls: 

    sudo sysctl --system


## Firewall Rules

**`/etc/iptables/rules.v4`**

```
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
# ==> INPUT 
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -m comment --comment "Default deny rule" -j REJECT --reject-with icmp-host-unreachable
# ==> FORWARD 
-A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT -m comment --comment "Outside->Inside"
-A FORWARD -i eth1 -o eth0 -m state --state NEW,RELATED,ESTABLISHED -m comment --comment "Inside->Outside" -j ACCEPT
-A FORWARD -m comment --comment "Default deny rule" -j REJECT --reject-with icmp-host-unreachable
COMMIT
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A POSTROUTING -o eth0 -j MASQUERADE
COMMIT
```

Reload the rules: 

    sudo iptables-restore /etc/iptables/rules.v4

Check active rules

    sudo iptables -L -v


