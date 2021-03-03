# Gentoo: Rescue system

Enable sshd: 

```
passwd
service sshd start
```

SSH in. Set up user env. 



### Mount Disks

Open the CryptLVM device: 

```
cryptsetup open /dev/sda3 cryptlvm
```

mount the devices

```
swapon /dev/stor/swap 
mount /dev/stor/root /mnt/gentoo
mount /dev/stor/home /mnt/gentoo/home
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev 
```

Enter the chroot: 

```
chroot /mnt/gentoo /bin/bash 
source /etc/profile 
export PS1="(chroot) ${PS1}"
```

Add /boot

```
mount /dev/sda2 /boot
```

