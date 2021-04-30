# Attach Bridge to VM


## Create network bridge

First, set up the host system with a bridge. For Ubuntu: 

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: no
  bridges:
    br0:
      dhcp4: yes
      interfaces:
        - eno1
```

## Attach bridge to running VM

Attach br0 to an existing VM, with a randomized MAC address: 

    virsh attach-interface --domain my-cool-vm --type bridge \
      --source br0 --model virtio --config --live \
      --mac $(openssl rand -hex 6 |sed 's/^\(..\)\(..\)\(..\)\(..\)\(..\).*$/02:\1:\2:\3:\4:\5/')

The bridge will immediately appear as a network interface in the vm, for example `enp6s0`. 
