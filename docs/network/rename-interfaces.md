# Rename a Network Interface

If you're sick of interface names like "enp0s1" and yearn for the good old days of "eth0"

Find the MAC address of the interface. 

```
ip link
```

Create a file at `/etc/systemd/network/10-persistent-eth0.link`

```
[Match]
MACAddress=AA:BB:CC:DD:00:11

[Link]
Name=eth0
```

Restart the box. Once back up, the interface will be renamed to "eth0", the way it should be. 

