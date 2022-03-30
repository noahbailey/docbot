# NetFilter / NFT

NetFilter is a 'modern' replacement for IPTables. 

## Install

    sudo apt install nftables

## Config

A simple workstation config: `/etc/nftables.conf`

```sh
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
        chain input {
                type filter hook input priority 0;

                #Statful connections
                ct state {established, related} accept
                ct state invalid drop
                #Accept loopback
                iifname lo accept

                ip protocol icmp accept
                ip6 nexthdr icmpv6 accept

                tcp dport {ssh} accept

                #Drop all other traffic
                reject with icmp type port-unreachable
                
        }
        chain forward {
                type filter hook forward priority 0;
                drop
        }
        chain output {
                type filter hook output priority 0;
        }
}
```

Load the config:

    sudo nft -f /etc/nftables.conf

### View the running config: 

    sudo nft list table inet filter

