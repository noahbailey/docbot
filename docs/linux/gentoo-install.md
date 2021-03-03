# Gentoo: Installation

## Installing Operating System

Check date and time are correct: (It should be UTC!)

    date

Sync NTP

    ntpd -q -g

### Downloading the stage tarball

    cd /mnt/gentoo

Select a stage 3 tarball from the [Gentoo site](https://www.gentoo.org/downloads/#other-arches)

* Should generally use a multilib (32/64) tarball
* Can use systemd instead of openrc

Download the tarball _and_ the signature: 

    wget http://distfiles.gentoo.org/releases/amd64/autobuilds/20190522T214502Z/stage3-amd64-20190522T214502Z.tar.xz
    
    wget http://distfiles.gentoo.org/releases/amd64/autobuilds/20190522T214502Z/stage3-amd64-20190522T214502Z.tar.xz.DIGESTS.asc

Verify the signatures: 

    gpg --keyserver hkps.pool.sks-keyservers.net --recv-keys BB572E0E2D182910 
    
    gpg --verify stage3-amd64~~~.tar.xz.DIGESTS.asc
    
    sha512sum -c stage3-amd64~~~.tar.xz.DIGESTS.asc 

### Extract the tarball

Extract the tarball to the root directory of the install: 

    cd /mnt/gentoo
    tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner


### Configure environment 

Modify the compile flags: 

##### /mnt/gentoo/etc/portage/make.conf

    # Compiler flags to set for all languages
    COMMON_FLAGS="-march=native -O2 -pipe"
    # Use the same settings for both variables
    CFLAGS="${COMMON_FLAGS}"
    CXXFLAGS="${COMMON_FLAGS}"

In the same file, set the number of threads to be used for compile, in this case a quad core system with SMT: 

    MAKEOPTS="-j9"

And configure a mirror

    mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
    
    mkdir --parents /mnt/gentoo/etc/portage/repos.conf
    cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

Copy DNS config to the chroot

    cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

Bind the pseudo-filesystems to the chroot: 

    mount --types proc /proc /mnt/gentoo/proc 
    mount --rbind /sys /mnt/gentoo/sys 
    mount --make-rslave /mnt/gentoo/sys 
    mount --rbind /dev /mnt/gentoo/dev 
    mount --make-rslave /mnt/gentoo/dev 

### Configure the system

    chroot /mnt/gentoo /bin/bash
    . /etc/profile
    export PS1="(chroot) ${PS1}"

#### Mount boot partition

    mkdir /boot
    mount /dev/sda2 /boot

Mount EFI partition

```
mkdir /boot/efi
mount /dev/sda1 /boot/efi
```



#### Configure portage

    emerge-webrsync 
    
    emerge --sync
    
    eselect profile list

Choose a profile that you like. I like Plasma and systemd because I am a massive soyboy beta. 

    eselect profile set default/linux/amd64/17.0/desktop/plasma/systemd
    
    emerge --ask --verbose --update --deep --newuse @world

Configure the timezone: 

    echo "America/Toronto" > /etc/timezone 
    emerge --config sys-libs/timezone-data

Configure the locale: 

    nano /etc/locale.gen
    
    > en_US ISO-8859-1
    > en_US.UTF-8 UTF8
    
    locale-gen
    locale -a

Set locale for package manager and refresh environment: 

    eselect locale list
    eselect locale set 7 (en_US.utf8)
    env-update && source /etc/profile && export PS1="(chroot) $PS1"

## Installing Sources

### The Kernel (Source)

    emerge --ask sys-kernel/gentoo-sources
    
    emerge --ask sys-apps/pciutils
    
    cd /usr/src/linux 
    make menuconfig

Follow the directions. 

Make sure the following pieces are compiled into the kernel: 

1. iwlfifi support
2. crypt support - https://wiki.gentoo.org/wiki/Dm-crypt

Now it's compiling time. 

    make && make modules_install
    
    make install

### The Kernel (Binary)

Simply install the distrokernel from portage: 

    emerge --ask sys-kernel/installkernel-gentoo
    emerge --ask sys-kernel/gentoo-kernel-bin

