# IP Blocklisting

## Install required software

    sudo apt install ipset ipset-persistent

## Configure blocklist

    sudo mkdir -p /var/lib/blocklist

### 'spamhaus' DROP list

**D**o **N**ot **R**oute or **P**eer

    curl https://www.spamhaus.org/drop/drop.txt | awk '/^[0-9]/{print $1}' | sudo tee /var/lib/blocklist/spamhaus.txt

Create ipset

    sudo /usr/sbin/ipset create spamhaus hash:net hashsize 8192

Populate the ipset with the blocklist file:

```sh
#!/bin/bash
/usr/sbin/ipset flush spamhaus
while read ip; do 
    /usr/sbin/ipset add spamhaus $ip -exist || echo $ip
done < /var/lib/blocklist/spamhaus.txt
```

This will take about a minute to run.

### abuse.ch c2 blocklist

Download the blocklist

    curl https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt | awk '/^[0-9]/{print $1}' | tr -d '\r' | sudo tee /var/lib/blocklist/abusech.txt

Create ipset

    sudo /usr/sbin/ipset create abusech hash:ip hashsize 4096

Populate the ipset with the c2 file:

```sh
#!/bin/bash
/usr/sbin/ipset flush abusech
while read ip; do 
    /usr/sbin/ipset add abusech $ip -exist || echo $ip
done < /var/lib/blocklist/abusech.txt
```


## Check the blocklist:

```
$ sudo /usr/sbin/ipset list -t

Name: spamhaus
Type: hash:net
Revision: 7
Header: family inet hashsize 8192 maxelem 65536 bucketsize 12 initval 0x68f90c54
Size in memory: 41520
References: 4
Number of entries: 900

Name: abusech
Type: hash:ip
Revision: 5
Header: family inet hashsize 4096 maxelem 65536 bucketsize 12 initval 0xe508fe5a
Size in memory: 9352
References: 0
Number of entries: 236
...
```

Check the blocklist by performing a lookup: 

```
$ sudo /usr/sbin/ipset test spamhaus 168.151.54.25

Warning: 168.151.54.25 is in set spamhaus.
```

## Iptables rules

Add an iptables rule to block ranges on the blocklist. 
For a server or workstation, this should probably go on the INPUT and OUTPUT chains:

```
-A INPUT  -m set --match-set spamhaus src -j LOG --log-prefix "BLOCK:SPAMHAUS:" --log-level 4
-A INPUT  -m set --match-set spamhaus src -j DROP -m comment --comment "drop droplisted IP ranges"
-A OUTPUT -m set --match-set spamhaus dst -j LOG --log-prefix "BLOCK:SPAMHAUS:" --log-level 4
-A OUTPUT -m set --match-set spamhaus dst -j DROP -m comment --comment "drop droplisted IP ranges"
-A OUTPUT -m set --match-set abusech  dst -j LOG --log-prefix "BLOCK:ABUSECH:" --log-level 4
-A OUTPUT -m set --match-set abusech  dst -j DROP -m comment --comment "drop droplisted IP ranges"
...
```

For a firewall, this should be added to FORWARD before any other rules that will take effect on the "outside" interface.

```
-A FORWARD -m set --match-set spamhaus src -j LOG --log-prefix "BLOCK:SPAMHAUS:" --log-level 4
-A FORWARD -m set --match-set spamhaus src -j DROP -m comment --comment "drop droplisted IP ranges"
-A FORWARD -m set --match-set spamhaus dst -j LOG --log-prefix "BLOCK:SPAMHAUS:" --log-level 4
-A FORWARD -m set --match-set spamhaus dst -j DROP -m comment --comment "drop droplisted IP ranges"
-A FORWARD -m set --match-set abusech  dst -j LOG --log-prefix "BLOCK:ABUSECH:" --log-level 4
-A FORWARD -m set --match-set abusech  dst -j DROP -m comment --comment "drop droplisted IP ranges"
```

Once set, the updated ruleset should be loaded:

    sudo iptables-restore < /etc/iptables/rules.v4

Once added to the active rules, it should appear like this: 

```
Chain OUTPUT (policy ACCEPT 65 packets, 8447 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set spamhaus dst LOG flags 0 level 4 prefix "BLOCK:SPAMHAUS:"
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set spamhaus dst /* drop droplisted IP ranges */
    4   336 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set abusech dst LOG flags 0 level 4 prefix "BLOCK:ABUSECH:"
    4   336 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set abusech dst /* drop droplisted IP ranges */
```

## Persistent ipset list

    # ipset save > /etc/ipset.conf

Enable the boot-up service

    sudo systemctl enable --now ipset.service

## Update scripts

`/opt/blocklist_spamhaus.sh`

```sh
#!/bin/bash

if [ $(id -u) -ne 0 ]; then 
    echo "!! Must be root" && exit 1
fi

curl -s https://www.spamhaus.org/drop/drop.txt | awk '/^[0-9]/{print $1}' > /var/lib/blocklist/spamhaus.txt

/usr/sbin/ipset flush spamhaus

while read ip; do 
    /usr/sbin/ipset add spamhaus $ip -exist || echo $ip
done < /var/lib/blocklist/spamhaus.txt

/usr/sbin/ipset save > /etc/ipset.conf
```

`/opt/blocklist_abusech.sh`

```sh
#!/bin/bash

if [ $(id -u) -ne 0 ]; then 
    echo "!! Must be root" && exit 1
fi

curl -s https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt | awk '/^[0-9]/{print $1}' | tr -d '\r' > /var/lib/blocklist/abusech.txt

/usr/sbin/ipset flush abusech

while read ip; do 
    /usr/sbin/ipset add abusech $ip -exist || echo $ip
done < /var/lib/blocklist/abusech.txt

/usr/sbin/ipset save > /etc/ipset.conf
```

`/etc/cron.d/droplist_update`

    # Weekly update for spamhaus list
    0 0 * * 1    root    /opt/blocklist_spamhaus.sh

    # Hourly update for abusech dynamic C2 list
    0 * * * *    root    /opt/blocklist_abusech.sh

## Nginx ban script

Create the initial ipset list:

    ipset create scanners hash:ip hashsize 4096

A script to generate blocked hosts based on their presence in a spam log:

```sh
/usr/sbin/ipset flush scanners
for ip in $(awk '{print $1}' < /var/log/nginx/spam.log | sort | uniq); do
    ipset add scanners $ip || echo $ip
done
/usr/sbin/ipset save > /etc/ipset.conf
```

Iptables rules to drop requests from these hosts:

```
-A INPUT -m set --match-set scanners src -j LOG --log-prefix "BLOCK:SCANNERS:" --log-level 4
-A INPUT -m set --match-set scanners src -j DROP -m comment --comment "repeat scanners"
```
