---
title: "Encrypted LVM/LUKS Linux Install"
description: "Install a secured Linux system using LUKS and LVM"
date: 2020-04-10T15:13:21-04:00
draft: false
---

This is a basic procedure for setting up a linux system my preferred way.

By no means is this a tutorial or a full guide to installing Linux. There are many parts that have been excluded, since this mainly focuses on the disk layout and partitioning.

Many parts are optional, and many partitions are not strictly required. For example, the ESP partition can be shared with the /boot partition. I don't like to do that, since that requires that the kernel be stored on FAT32 which is icky, but it does reduce the complexity of the install. Also, LVM volumes for `/var` and `/tmp` can also be safely excluded.

This setup works great for Arch Linux and Gentoo, and likely for Debian as well. 


## Set up disks

Assuming /dev/sda is our primary disk.

Disk Layout:

Partition | Purpose              | Label | Size
--------- | -------------------- | ----- | ----
/dev/sda1 | GRUB2 Bootloader     | grub  | 1 MB
/dev/sda2 | EFI System Partition | efi   | 128 MB
/dev/sda3 | initramfs + kernel   | boot  | 500 MB
/dev/sda4 | LUKS + LVM 	         | lvm   | 500 GB


The rest of the volumes will be set up using LVM slices inside the LUKS container.

### Build Partitions

```
parted -a optimal /dev/sda
	
	mklabel GPT
	mkpart primary 1 3 
	name 1 grub 
	set 1 bios_grub on 
	
	mkpart primary 3 131
	name 2 efi 
	set 2 boot on

	mkpart primry 131 631
	name 3 boot

	mkpart primary 631 100%
	name 4 lvm 

```

### Create LUKS volume

    cryptsetup luksFormat --type luks2 /dev/sda4
    cryptsetup open /dev/sda4 cryptlvm

### Create LVM device

    pvcreate /dev/mapper/cryptlvm
    vgcreate stor /dev/mapper/cryptlvm


## LVM Config

LVM is flexible, and more or less containers can be added. Note: I like to have a separate `/var/log` mount point to improve system stability. 

Volume Name | Mount Point | Size
----------- | ----------- | ----
swap	    | [swap] 	  | 4 GB
root	    | /		  | 60 GB
var	    | /var	  | 20 GB
log	    | /var/log	  | 5 GB
tmp	    | /tmp	  | 2 GB
home	    | /home	  | 360 GB

### Create Volumes

    lvcreate -L 4G  stor -n swap
    lvcreate -L 60G stor -n root
    lvcreate -L 20G stor -n var
    lvcreate -L 5G  stor -n log
    lvcreate -L 2G  stor -n tmp
    lvcreate -l 100%FREE stor -n home

### View LVM config

    vgs - show volume groups
    lvs - show logical volumes

## Format & Mount Volumes

Partition Path | Mount Point | Filesystem
-------------- | ----------- | ----------
/dev/sda2      | n/a	     | fat32
/dev/sda3      | /boot	     | ext2
/dev/stor/swap | [swap]	     | swap
/dev/stor/root | /	     | ext4
/dev/stor/var  | /var	     | ext4
/dev/stor/log  | /var/log    | ext4
/dev/stor/tmp  | /tmp	     | ext4
/dev/stor/home | /home	     | ext4


### Initialize Filesystems

    mkfs.fat -F 32 /dev/sda2
    mkfs.ext2 /dev/sda3
    mkfs.ext4 /dev/stor/{root,var,log,tmp,home}

### Set up Swap

    mkswap /dev/stor/swap
    swapon /dev/stor/swap

### Mount Filesystems

Substitue the mount point with where your distribution targets the install. Example, gentoo uses `/mnt/gentoo/`

Root filesystem:

    mount /dev/stor/root /mnt/

Var and Log filesystems:

    mkdir -p /mnt/var/log
    mount /dev/stor/var /mnt/var/
    mount /dev/stor/log /mnt/var/log

Temp filesystem

     mkdir -p /mnt/tmp
     mount -o nodev,noexec /dev/stor/tmp /mnt/tmp

Home filesystem

     mkdir -p /mnt/home
     mount /dev/stor/home /mnt/home



## Bootloader and Initramfs

I prefer grub because it's boring. Other bootloaders can be used as well.

### GRUB

Afer building/installing the `grub2` binaries on the target systems, the ESP partition can be set up.

     mkdir -p /boot/efi
     mount /dev/sda2 /boot/efi

Then `grub2` can be installed into the ESP.

     grub-install -target=x86_64-efi --efi-directory=/boot/efi

The ESP can be unmounted after this. It generally does not need to be mounted.

    umount /boot/efi

### GRUB config

Edit the `/etc/default/grub` and add cryptlvm to the boot.

This may vary based on GRUB version and distribution. For Arch linux:

     GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda4:cryptlvm rw quiet"

Or, for Gentoo:

     GRUB_CMDLINE_LINUX="crypt_root=/dev/sda4 init=/lib/systemd/systemd dolvm"

Then, the config is set up.

      grub-mkconfig -o /boot/grub/grub.cfg

### Archlinux initcpio

For Arch, the simplest way to go is to use initcpio.

`/etc/mkinitcpio.conf`

	HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt lvm2 filesystems fsck)

And then generate the initramfs:

    mkinitcpio -p linux

### Genkernel intramfs

Make sure that genkernel-next is installed. The `cryptsetup` USE flag must also be used.

`/etc/portage/package.use`

     sys-kernel/genkernel-next  cryptsetup

Then genkernel is reconfigured.

`/etc/genkernel.conf`

	LVM="yes"
	LUKS="yes"
	UDEV="yes"
	BOOTLOADER="grub"

And a new initramfs is compiled and installed.

      genkernel initramfs

## Services

Make sure that UDEV and LVMETAD are running on boot.

Systemd:

	systemctl enable lvm2-lvmetad

OpenRC:

	rc-update add lvmetad boot


## System Rescue

If the system doesn't boot correctly, you can get back in from the grub emergency shell.

       cryptsetup open /dev/sda4 cryptlvm

       > luks passphrase...

       mount /dev/stor/root /new_root

Then the kernel will pick up from where it left off, and start the init system... Hopefully getting you back up and running.

