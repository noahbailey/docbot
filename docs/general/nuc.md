# NUC on Linux

## NIC driver 

NUC6CAYH has issues with the regular kernel driver. Install the DKMS driver for the NIC: 

    sudo apt install r8168-dkms


```
Backing up initrd.img-5.10.0-9-amd64 to /boot/initrd.img-5.10.0-9-amd64.old-dkms
Making new initrd.img-5.10.0-9-amd64
(If next boot fails, revert to initrd.img-5.10.0-9-amd64.old-dkms image)
update-initramfs.........
```

## Microcode updates

Make sure you have non-free sources enabled first. 

```
deb http://deb.debian.org/debian bullseye main contrib non-free
deb http://security.debian.org/debian-security bullseye-security main contrib non-free
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
```

Then install the microcode update package: 

    apt install intel-microcode

Then, reboot the host. 
