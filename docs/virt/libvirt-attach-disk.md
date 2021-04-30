# Attach disk to VM

## Create a disk file

Create a qcow image: 

    qemu-img create -f qcow2  -o preallocation=metadata disk-vdb.qcow2 50G

View the qcow image: 

```
$ qemu-img info ./disk-vdb.qcow2 

image: ./disk-vdb.qcow2
file format: qcow2
virtual size: 50 GiB (53687091200 bytes)
disk size: 8.01 MiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

```

Note: with the `preallocation=metadata` mode of thin-provisioning, the disk image will appear very large but it is a sparse image. On a newly created disk, note the discrepancy between `ls` and `du`: 

```
$ ls -lh disk-vdb.qcow2 
-rw-r--r-- 1 ongo ongo 51G Apr 30 17:22 disk-vdb.qcow2

$ du -h ./disk-vdb.qcow2 
8.1M    ./disk-vdb.qcow2
```

Over time the disk file will grow and shrink as its usage inside the VM fluctuates. 

## Attach disk to VM

    virsh attach-disk server-01 --source /path/to/disk-vdb.qcow2 --target vdb --driver qemu

## Partition and format disk

On the VM, view `lsblk` to find the disk path: 

```
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     252:0    0   60G  0 disk 
├─vda1  252:1    0 59.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0   50G  0 disk                    <-- THIS DISK
```

Create a GPT partition table on the disk: 

    sudo parted -a opt /dev/vdb mklabel gpt
    sudo parted /dev/vdb mkpart primary ext4 0% 100%

Create a single EXT4 filesystem on the disk, with the label "datavol": 

    sudo mkfs.ext4 -L datavol /dev/vdb

## Mount the EXT4 filesystem

Create a mountpoint

    sudo mkir -p /mnt/disk-vdb

Find the filesystem UUID, and write an `/etc/fstab` entry: 

    UUID=02bb033c-a191-40c0-94fb-bb025c724b13  /mnt/disk-vdb   ext4  defaults,discard  0  1

Mount all filesystems: 

    sudo mount -a

## Detach disk from VM

Disks can be detached in a similar way: 

    virsh detach-disk server-01 --target vdb