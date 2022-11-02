# IP Blocklisting

## Install required software

    sudo apt install ipset ipset-persistent

## Configure blocklist

    sudo mkdir -p /var/lib/blocklist

    curl https://www.spamhaus.org/drop/drop.txt | awk '/^[0-9]/{print $1}' | sudo tee /var/lib/blocklist/drop.txt

Create ipset

    sudo /usr/sbin/ipset create blocklist hash:net hashsize 8192

    sudo /usr/sbin/ipset flush  blocklist

A script to populate ipset with the drop list:

```sh
#!/bin/bash
while read ip; do 
    /usr/sbin/ipset add blocklist $ip -exist || echo $ip
done < /var/lib/blocklist/drop.txt
```

This will take about a minute to run. Once complete, the blocklist can be viewed:


```
$ sudo /usr/sbin/ipset list blocklist

Name: blocklist
Type: hash:net
Revision: 7
Header: family inet hashsize 8192 maxelem 65536 bucketsize 12 initval 0xf43d9e99
Size in memory: 41328
References: 0
Number of entries: 900
Members:
168.151.54.0/24
199.26.137.0/24
...
```

Check the blocklist by performing a lookup: 

```
$ sudo /usr/sbin/ipset test blocklist 168.151.54.25

Warning: 168.151.54.25 is in set blocklist.
```

## Iptables rules

Add an iptables rule to block ranges on the blocklist. 
For a server or workstation, this should probably go on the INPUT and OUTPUT chains:

```
-A INPUT  -m set --match-set blocklist src -j LOG --log-prefix "BLOCKLIST:" --log-level 4
-A INPUT  -m set --match-set blocklist src -j DROP -m comment --comment "drop droplisted IP ranges"
-A OUTPUT -m set --match-set blocklist dst -j LOG --log-prefix "BLOCKLIST:" --log-level 4
-A OUTPUT -m set --match-set blocklist dst -j DROP -m comment --comment "drop droplisted IP ranges"
...
```

For a firewall, this should be added to FORWARD before any other rules that will take effect on the "outside" interface.

```
-A FORWARD -m set --match-set blocklist src -j LOG --log-prefix "BLOCKLIST:" --log-level 4
-A FORWARD -m set --match-set blocklist src -j DROP -m comment --comment "drop droplisted IP ranges"
-A FORWARD -m set --match-set blocklist dst -j LOG --log-prefix "BLOCKLIST:" --log-level 4
-A FORWARD -m set --match-set blocklist dst -j DROP -m comment --comment "drop droplisted IP ranges"
```

Once set, the updated ruleset should be loaded:

    sudo iptables-restore < /etc/iptables/rules.v4

Once added to the active rules, it should appear like this: 

```
 pkts bytes target     prot opt in     out     source               destination         
    0     0 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set blocklist src LOG flags 0 level 4 prefix "BLOCKLIST:"
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set blocklist src /* drop droplisted IP ranges */
  171 31280 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
```

## Persistent ipset list

    # ipset save > /etc/ipset.conf

Enable the boot-up service

    sudo systemctl enable --now ipset.service

## Update script

`/opt/blocklist.sh`

```sh
#!/bin/bash

if [ $(id -u) -ne 0 ]; then 
    echo "!! Must be root" && exit 1
fi

curl -s https://www.spamhaus.org/drop/drop.txt | awk '/^[0-9]/{print $1}' > /var/lib/blocklist/drop.txt

while read ip; do 
    /usr/sbin/ipset add blocklist $ip -exist || echo $ip
done < /var/lib/blocklist/drop.txt

ipset save > /etc/ipset.conf
```

`/etc/cron.d/droplist_update`

    0 0 * * 1    root    /opt/blocklist.sh
