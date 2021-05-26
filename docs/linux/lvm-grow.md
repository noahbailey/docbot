# Grow LVM partition

Most commonly, this is because you've added storage in VMware (or other hypervisor) and need it available within the VM...

## Grow the partition 

Expand the GPT partition. For NVMe:

    sudo growpart /dev/nvme0n1 1

Or for SATA: 

    sudo growpart /dev/sda 1

Or for Libvirt: 

    sudo growpart /dev/vda 1

## Expand the LVM Physical Volume

Grow the space that is accessible to LVM: 

    sudo pvresize /dev/vda1

## Expand the LVM Volume and Filesystem

Expand the volume used for `/home`: 

    sudo lvextend -l +100%FREE /dev/stor/home --resizefs

Expand the volume for `/` (root): 

    sudo lvextend -l +100%FREE /dev/stor/root --resizefs

Expand the volume for `/var` by only 10 GB: 

    sudo lvextend -L +10G /dev/stor/var --resizefs

