---
title: 2. System Configuration
---

# Setup the system

Afer the kernel is built, the system needs config. 

### Utilities

```
emerge --ask --quiet sys-fs/cryptsetup
emerge --ask --quiet sys-fs/lvm2 
```



### FSTAB

Set up the FSTAB: 

```fstab
/dev/sda2               /boot           ext2            noauto,noatime  1 2

UUID=890af2cc-bf23-4fc5-b890-06c97d714547   none   swap   sw 0 0

# LVM group
UUID=0f57be29-d1b7-4e2d-b3fa-ef8c66c6a982   /      ext4   defaults  0  1
UUID=5685ac4d-2a52-4ff0-b57c-a9baee0af608   /var   ext4   defaults  0  0
UUID=fb76e7ee-df9f-41a5-b28c-e941348483d1   /var/log  ext4  defaults 0  0
UUID=4cc4f511-916e-4a87-ad52-6c83d55901a7   /home  ext4   defaults  0  0

# /tmp
UUID=9add4878-d741-400a-b4b3-793783fa4bf1   /tmp   ext4   defaults,noexec,nodev  0 0

```



### Filesystem Tools

Tools for `ext4` formatted drives: 

```
emerge --ask  sys-fs/e2fsprogs
```

Userspace componenets for `lvm2`

```
emerge --ask sys-fs/lvm2

rc-update add lvm boot
rc-update add device-mapper boot
```



### Initramfs (dracut)

```
emerge -a sys-kernel/dracut
```

Create the initramfs

```
dracut --hostonly --kver 5.4.28-gentoo --lvm --force
```



### Initramfs (genkernel)

IMPORTANT: must use genkernel-next!

```
genkernel --install --luks --lvm --no-ramdisk-modules initramfs
```

--keymap



```
* - Add "dolvm" for LVM support
* - Add "crypt_root=<device>" for LUKS-encrypted root
* - Add "crypt_swap=<device>" for LUKS-encrypted swap
```



## Networking

Wireless

```
emerge --ask net-wireless/iw net-wireless/wpa_supplicant
```

DHCP 

```
emerge --ask net-misc/dhcpcd
```

Network manager

```
net-misc/networkmanager
```



## Bootloader - Install

```
emerge --ask --verbose sys-boot/os-prober
```



```
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask --verbose sys-boot/grub:2
```

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi
```



## Bootloader - Configure

Setup grub: 

```
GRUB_CMDLINE_LINUX="cryptdevice=UUID=f23f90b1-3df9-4467-a616-a09ff382a85b:cryptlvm resume=/dev/mapper/computer-swap root=/dev/mapper/computer-root rw quiet"
```

:information_source: The cryptdevice UUID must be the id of the `/dev/sda3` partition, NOT the LVM PV. 



```
genkernel --luks --lvm --menuconfig all
```

