# KVM and Libvirt


## Install 

    sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
    sudo systemctl enable libvirtd
    sudo systemctl start libvirtd

Additional utilities: 

    sudo apt install cloud-utils virtinst

Add your user to the `libvirt` group: 

    sudo useradd $USER libvirt

## Bridge

`/etc/netplan/20-kvm-config.yml`

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      dhcp6: no

  bridges:
    br0:
    interfaces: [ens33]
      addresses:
        - 10.20.10.31/24
      gateway4: 10.20.10.2
      nameservers:
        search:
          - intranet.mycooldomain.com
        addresses:
          - 10.20.10.11
          - 10.20.10.12
```

Test and apply the configuration. 

    sudo netplan generate
    sudo netplan apply

## Cloud Image

Cloud Image config for 18.04

    wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
    qemu-img info bionic-server-cloudimg-amd64.img 

Convert to QCOW: 

    sudo qemu-img convert -f qcow2 bionic-server-cloudimg-amd64.img /virt/templates/bionic-server-cloudimg-amd64.img

Clone the VM image: 

    qemu-img create -f qcow2 -b /virt/templates/bionic-server-cloudimg-amd64.img /virt/virtualmachines/virt-01.img


Create a cloud-config template: 

```yml
#cloud-config
password: not-your-password
chpasswd: { expire: False }
ssh_pwauth: True
hostname: virt-01
ssh_authorized_keys: 
  - "https://github.com/example.keys"
```

Generate a cloud-config boot disk: 


    sudo apt install cloud-image-utils
    sudo cloud-localds /virt/config/virt-01_cloudconfig.img /virt/config/virt-01_cloudconfig.yml

## Create a VM

```bash
virt-install --name virt-01 --memory 512 --vcpus 1 \
    --disk /virt/vms/virt-01.img,device=disk,bus=virtio \
    --disk /virt/config/virt-01_cloudconfig.img,device=cdrom \
    --os-type linux --os-variant ubuntu18.04 \
    --virt-type kvm --graphics none \
    --network network=default,model=virtio --import
```
