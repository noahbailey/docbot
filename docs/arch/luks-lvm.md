---
title: Full Disk Encryption
description: 
published: true
date: 2019-08-03T00:51:02.274Z
tags: 
---

# LUKS and LVM

## Partition Layout

    | <-------> | <---------------------> |
    | /dev/sda1 | /dev/sda2               |
    | /boot     | cryptlvm (LUKS)         |
    |           |    /dev/stor/root   | 
    |           |    /dev/stor/home   | 
    | <-512M--> | <---------118G--------> |

## Encrypted Volumes

Create the LUKS volume: 

    cryptsetup luksFormat --type luks2 /dev/sda2
    cryptsetup open /dev/sda2 cryptlvm

Create the LVM physical device and volume group: 

    pvcreate /dev/mapper/cryptlvm
    vgcreate stor /dev/mapper/cryptlvm

## LVM volumes

Create the volumes for swap, root, and home: 

    lvcreate -L 4G stor -n swap 
    lvcreate -L 25G stor -n root 
    lvcreate -l 100%FREE stor -n home

Format the volumes as ext4: 

    mkfs.ext4 /dev/stor/root
    mkfs.ext4 /dev/stor/home
    mkswap /dev/stor/swap 

Mount the volumes at /mnt so the system may be installed: 

    mount /dev/stor/root /mnt
    mkdir -p /mnt/home
    mount /dev/stor/home /mnt/home
    swapon /dev/stor/swap 

### Viewing LVM 

LVM volumes can be viewed with the following shell commands: 

    vgs - show volume groups
    lvs - show logical volumes

## Boot partition 

The /boot partition is not encrypted, and exists as a 512MB volume at the beginning of the device. 

    mkfs.ext4 /dev/sda1
    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot

## Installing Arch 

Arch linux is installed per the usual procedure: 

    pacstrap /mnt base base-devel
    
    genfstab -pU /mnt >> /mnt/etc/fstab


## Configuring Bootloader

First, grub2 is installed: 

    pacman -S grub

A critical element of the system is ensuring that `grub` is able to find the encrypted volume such that the initramfs kernel can prompt for password. 

To accomplish this, the UUID of crypto_LUKS volume must be in the grub configuration file. 

This is the output of an example `lsblk -f` to list the filesystem hierarchy and UUIDs:

```
NAME                FSTYPE      LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT
sda                                                                                         
|-sda1              ext4              e6564bee-b5bc-45dc-a46e-3b0b1ca9394b    376.1M    15% /boot
`-sda2              crypto_LUKS       f23f90b1-3df9-4467-a616-a09ff382a85b                  
  `-cryptlvm        LVM2_member       0H3W9J-Oe8H-G37X-m0p8-lBRc-ViAr-hzvT8T                
    |-stor-swap swap              2982187e-124a-4067-af78-68fbcab41883                  [SWAP]
    |-stor-root ext4              3f202605-e8da-4a78-aa7a-07c922d5b731     17.5G    24% /
    `-stor-home ext4              2f1f7db1-9034-4a1d-8e28-cf05b061027a
```

In the grub configuration file, the UUID is set: 

#### /etc/default/grub  
    GRUB_CMDLINE_LINUX="cryptdevice=UUID=f23f90b1-3df9-4467-a616-a09ff382a85b:cryptlvm resume=/dev/mapper/stor-swap root=/dev/mapper/stor-root rw quiet"

Then, the `initramfs` is configured to correctly hook LUKS: 

#### /etc/mkinitcpio.conf
    HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt lvm2 filesystems fsck)

## Generating grub config

Because of a bug in recent versions of `udev` (cough cough Lennart), it may be nessicary to mount the non-chroot /run inside the chroot such that it can find the LVM volume UUIDs during config generation. 

#### non-chroot

    mkdir /mnt/hostrun
    mount --bind /run /mnt/hostrun

#### chroot

    mkdir /run/lvm
    mount --bind /hostrun/lvm /run/lvm

Then, grub may be installed into the MBR: 

    grub-install --target=i386-pc /dev/sda
    grub-mkconfig -o /boot/grub/grub.cfg

Finally, the initramfs can be generated. 

    mkinitcpio -p linux

Assuming all other installation steps have been taken, the system may be rebooted at this point. 


## Grub emergency shell

If the system is unable to find the `cryptlvm` volume, it will drop to emergency shell during boot. It may be possible to boot the system by manually decrypting the root volume: 

    cryptsetup open /dev/sda2 cryptlvm
    
    mount /dev/stor/root /new_root
    
    exit 

If this fails, other action must be taken to repair the system. 