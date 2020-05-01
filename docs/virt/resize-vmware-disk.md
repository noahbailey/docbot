---
title: Resize VMware Disk
---

# Resize VMware disk

Should be installed by default, but make sure it's present on the system. 

    sudo apt install cloud-guest-utils

Instruct the Linux kernel to rescan the block device
    
    sudo sh "echo 1 > /sys/class/block/sda/device/rescan"


Extend the GUID partition
    
    sudo growpart /dev/sda 1

### Partition

Resize the ext4 filesystem

    sudo resize2fs /dev/sda1 

### LVM

Resize the LVM physical volume to take up the rest of the disk
    
    sudo pvresize /dev/sda1

Extend the LVM volume that occupies that partition
    
    sudo lvextend -l +100%Free /dev/vg01/dbvol

Extend the overlayed EXT4 filesystem
    
    sudo resize2fs /dev/vg01/dbvol


