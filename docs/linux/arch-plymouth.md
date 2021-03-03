# Arch: Plymouth

Plymouth can be used to 'pretty up' the boot process on Linux. 

References: 

* [https://wiki.archlinux.org/index.php/Plymouth](https://wiki.archlinux.org/index.php/Plymouth)
* [https://wiki.archlinux.org/index.php/Mkinitcpio](https://wiki.archlinux.org/index.php/Mkinitcpio)
* [https://wiki.archlinux.org/index.php/Arch_boot_process#initramfs](https://wiki.archlinux.org/index.php/Arch_boot_process#initramfs)


## Install

Plymouth is only available through the AUR. 

    yay -S plymouth 

## Initramfs

Since plymouth loads before the kernel, it must be added to the initramfs. So, edit the `/etc/mkinitcpio.conf` config file, and change the hooks to the following: 

    HOOKS=(base udev plymouth autodetect keyboard keymap consolefont modconf block plymouth-encrypt lvm2 filesystems fsck)

It's also a good idea to make sure the display driver is injected into the initramfs: 

    MODULES=(ext4 i915)


## Theme

The theme can be selected in /etc/plymouth/plymouthd.conf

    sudo plymouth-set-default-theme -R spinner

## Display Manager

Enable smooth-transition by using the plymouth variat of the display manager: 

    sudo systemctl disable sddm
    sudo systemctl enable  sddm-plymouth


