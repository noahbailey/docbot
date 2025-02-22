# Simulate a bad connection

Use `tc` to enshittify your connection:

```
tc qdisc replace dev ens2 root netem corrupt 10%
tc qdisc change dev ens2 root netem delay 250ms 250ms distribution normal
```

If this doesn't work, check that the kernel module is loaded:

```
modprobe sch_netem
```

Or, use iptables:

```
iptables -A INPUT -i ens2 -m statistic --mode random --probability 0.05 -j DROP
```

## Bi-directional config with ifb device

`tc` will only effect egress traffic on an interface. The above config will slow down the downstream network connection but upload will be largely unaffected. 

Load the ifb module and create a device

```
modprobe ifb
ip link add name ifb1 type ifb
ip link set dev ifb1 up
```

Mirror the traffic from the interface. 

```
tc qdisc add dev eth1 handle ffff: ingress
tc filter add dev eth1 parent ffff: protocol ip u32 match u32 0 0 action mirred egress redirect dev ifb1
```

Apply bandwidth limitation (50mb/s) in both directions:

```
tc qdisc add dev eth1 root handle 1: htb default 10
tc class add dev eth1 parent 1:  classid 1:1 htb rate 50mbit
tc class add dev eth1 parent 1:1 classid 1:10 htb rate 50mbit
tc qdisc add dev ifb1 root handle 1: htb default 10
tc class add dev ifb1 parent 1:  classid 1:1 htb rate 50mbit
tc class add dev ifb1 parent 1:1 classid 1:10 htb rate 50mbit
```

Apply delays and corruption:

```
tc qdisc change dev eth1 root netem corrupt 0.1%
tc qdisc change dev ifb1 root netem loss 0.1%
tc qdisc change dev eth1 root netem delay 20ms 5ms distribution normal
tc qdisc change dev ifb1 root netem delay 20ms 5ms distribution normal
```


