# KVM and Qemu

## Install

    sudo pacman -S qemu libvirt virt-manager qemu-arch-extra virt-install ebtables dmidecode

### Activate service

    sudo systemctl enable --now libvirt.service

### User permission

Add your user to the `libvirt` group: 

    sudo usermod -aG libvirt ongo
