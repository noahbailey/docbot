# VLAN

## Install package

sudo apt install vlan

sudo modprobe 8021q

```
auto enp3s0.10
iface enp3s0.10 inet static
	address 10.204.10.20/24
	gateway 10.204.10.1
	vlan-raw-device enp3s0
```