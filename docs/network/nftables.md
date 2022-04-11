# NetFilter / NFT

NetFilter is a 'modern' replacement for IPTables. You can manipulate and modify nftables rules using the `nft` utility. 

## Install

    sudo apt install nftables

## Config

An example nftables config for my workstation, with stateful firewall and protocol counters: `/etc/nftables.conf`

```sh
#!/usr/sbin/nft -f

flush ruleset

table inet filter {

        # Traffic counters for each "significant" traffic flow
        counter cnt_stfl_in {}
        counter cnt_ssh_in {}
        counter cnt_reject {}
        counter cnt_output {}

        chain input {
                type filter hook input priority 0;

                #Stateful connections
                ct state {established, related} counter name cnt_stfl_in
                ct state {established, related} accept
                ct state invalid drop
                iifname lo accept

                ip protocol icmp accept
                ip6 nexthdr icmpv6 accept

                # Accept SSH traffic
                tcp dport {ssh} counter name cnt_ssh_in
                tcp dport {ssh} accept

                #Drop all other traffic
                counter name cnt_reject
                reject with icmp type port-unreachable
                
        }
        chain forward {
                type filter hook forward priority 0;
                drop
        }
        chain output {
                type filter hook output priority 0;
                counter name cnt_output
        }
}
```

Load the config:

    sudo nft -f /etc/nftables.conf

### View the running config: 

Show the filter table:

    sudo nft list table inet filter


Show all rulesets: 

    sudo nft list ruleset


## Counters

Note the lines in the original config with `counter name foobar` in them - these lines will increase a traffic counter any time they are matched. 

View the traffic counters: 

    sudo nft list counters

```
table inet filter {
	counter cnt_stfl_in {
		packets 51122 bytes 157539019
	}
	counter cnt_ssh_in {
		packets 0 bytes 0
	}
	counter cnt_reject {
		packets 154 bytes 21229
	}
	counter cnt_output {
		packets 35806 bytes 100711748
	}
}
```

Expose traffic counter data in JSON format: 

    sudo nft -j list counters

```json
{
  "nftables": [
    {
      "metainfo": {
        "version": "0.9.8",
        "release_name": "E.D.S.",
        "json_schema_version": 1
      }
    },
    {
      "counter": {
        "family": "inet",
        "name": "cnt_stfl_in",
        "table": "filter",
        "handle": 4,
        "packets": 51205,
        "bytes": 157560843
      }
    },
    {
      "counter": {
        "family": "inet",
        "name": "cnt_ssh_in",
        "table": "filter",
        "handle": 5,
        "packets": 0,
        "bytes": 0
      }
    },
    {
      "counter": {
        "family": "inet",
        "name": "cnt_reject",
        "table": "filter",
        "handle": 6,
        "packets": 157,
        "bytes": 21445
      }
    },
    {
      "counter": {
        "family": "inet",
        "name": "cnt_output",
        "table": "filter",
        "handle": 7,
        "packets": 35885,
        "bytes": 100727274
      }
    }
  ]
}
```