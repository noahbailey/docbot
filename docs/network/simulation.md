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
