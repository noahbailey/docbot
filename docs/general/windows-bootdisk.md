# Burn a Windows ISO on Linux

First of all, don't do this. Windows is dumb and if you use it you are also dumb (by the transitive property of course).

## Format & Mount USB 

Create the partition: 

    parted -a optimal /dev/sdb

        mklabel GPT

        mkpart primary 1 100%
        name 1 windows
        set 1 boot on 

Create the volume: 

    sudo mkfs.ntfs --quick /dev/sdb1

Mount the USB: 

    sudo mkdir -p /mnt/usb

## Mount the ISO

    sudo mkdir -p /mnt/iso

    sudo mount -o loop /home/ongo/iso/windows-9.iso /mnt/iso

## Copy files to USB

    sudo rsync -av --progress /mnt/iso/ /mnt/usb/
