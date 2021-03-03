# Gentoo: Binary Kernel

If you're impatient (or just have a slow CPU) you may want to skip compiling the kernel. 

## Config

For a LUKS/LVM install the following USE flags must be enabled globally: 

* `initramfs`
* `cryptsetup`

## Install

    emerge -a sys-kernel/gentoo-kernel-bin 

## GRUB config

This kernel style uses Dracut to build the initramfs, some modifications to the grub config are needed. 

`/etc/default/grub`

```
GRUB_CMDLINE_LINUX="root=UUID=<id-of-/dev/mapper/stor-root> rd.luks.uuid=<id-of-/dev/sda3> rd.lvm.vg=stor"
```

## Post Install

Rebuild modules & initramfs

    emerge --ask @module-rebuild
    emerge --config sys-kernel/gentoo-kernel-bin

