# Gentoo Maintenance

* https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Finalizing
* https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet



## Package Management

### Install a package

```
emerge --ask --verbose class/name
```

### System Updates

First, sync the updated source def tree

```
sudo emerge --sync
```

Then, package sources can be updated and built. 

```
emerge -uDU --keep-going --with-bdeps=y @world --ask
```

Sometimes, portage itself needs an update

```
sudo emerge --oneshot portage
```

To recompile preserved packges: 

```
emerge @preserved-rebuild
```



### System Upgrades

Upgrading an entire system with a new gentoo profile.

In this case, moving from 17.0 to 17.1

```
sudo eselect profile set default/linux/amd64/17.1

sudo emerge --update --deep --newuse -av @world 
```

Then recompile the packages that were held back. 

```
sudo emerge @preserved-rebuild
```

After the system is done recompiling, clean up the old versions

```
sudo emerge --depclean -a
. /etc/profile
```

Finally, rebuild the initramfs

```
genkernel --lvm initramfs
```





### Upgrade the Kernel

* https://wiki.gentoo.org/wiki/Kernel/Upgrade

Backup the kernel config before trying to upgrade. 

```
sudo cp /usr/src/linux/.config ~/kernel-config-`uname -r`
```

Allow 'unstable' packages: 

```
echo 'ACCEPT_KEYWORDS="~amd64"' \
	| sudo tee -a /etc/portage/make.conf
```



Download new kernel sources

```
sudo emerge --ask  sys-kernel/gentoo-sources:5.4.2
```

Select the kernel version to build

```
sudo eselect kernel list
sudo eselect kernel set 2
```

Copy the config file backup to the updated kernel

```
sudo cp kernel-config-4.19.86-gentoo /usr/src/linux/.config
```

Update the kernel config file 

```
cd /usr/src/linux
sudo make syncconfig 
```

Build the kernel 

```
sudo genkernel all 
```

After the build completes (~1h), update the grub config

```
grub-mkconfig -o /boot/grub/grub.cfg
```

And reconfigure the LVM boot config

```
genkernel --lvm initramfs
```



### Manual Kernel Upgrade

Faster than the automatic method. 

```
sudo cp /usr/src/linux/.config ~/kernel-config-`uname -r`
```



```
sudo eselect kernel list
sudo eselect kernel set 2
sudo make syncconfig 
sudo make -j4
sudo make modules_prepare
sudo make modules_install
sudo make install 

sudo dracut --hostonly --kver 5.4.11-gentoo-x86_64

sudo grub-mkconfig -o /boot/grub/grub.cfg
```



### Remove an old kernel

Remove the sources for old kernels

```
emerge --ask --depclean gentoo-sources
```

Remove all but the newest three kernels from /boot

```
eclean-kernel -n 3
```



## System Upgrades



## Users

Add a user

```
useradd -m -G users,wheel,audio -s /bin/bash noah
passwd noah
```



### Sudo

Install sudo

```
#emerge --config =mail-mta/nullmailer-2.2
emerge --ask app-admin/sudo
```

Grant root priv to members of `%wheel` group. 

```
echo "%wheel  ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/wheel
```



### SSH keys

Login as your new user, add your SSH keys from github. 

```
curl -s https://github.com/noahbailey.keys >> .ssh/authorized_keys
```

Or, copy them from another machine

```
ssh-copy-id user@host
```



## Comforts

### Text editors

GNU Emacs

```
sudo emerge --ask app-editors/emacs
```

Vim

```
sudo emerge --ask app-editors/vim 
```

Log output formatter for logs, etc

```
sudo emerge --ask app-misc/grc

```



### System Utils

Htop, tmux, etc

```
sudo emerge --ask app-misc/tmux
sudo emerge --ask net-misc/whois
sudo emerge --ask net-dns/bind-tools
sudo emerge --ask net-misc/rsync
```

### Maintenance Tools

* https://wiki.gentoo.org/wiki/Q_applets
* https://wiki.gentoo.org/wiki/Genlop



### Cleaning Up

Install cleanup tools

```
sudo emerge --ask app-admin/eclean-kernel
sudo emerge --ask app-portage/gentoolkit
```

Run the cleanup programs: 

```
sudo eclean-dist
sudo eclean-pkg
sudo eclean-kernel
```

### Monitoring tools

```
sudo emerge --ask net-analyzer/nload
sudo emerge --ask sys-process/htop
```

### Dev Tools

```
sudo emerge --ask dev-vcs/git
```

### Security Tools

VPN Clients/Servers

```
sudo emerge --ask net-vpn/openvpn
sudo emerge --ask net-vpn/wireguard
```

Host firewall

```
sudo emerge --ask net-firewall/ufw

sudo /etc/init.d/ufw start
sudo ufw allow ssh
sudo rc-update add ufw default
```

Networking and traffic capture tools

```
sudo emerge --ask net-analyzer/nmap
```



## Packages

* https://packages.gentoo.org/



## Desktop

### Setup user environment

```
sudo emerge --ask x11-misc/xdg-user-dirs

sudo emerge --ask games-misc/cowsay
```





### Config for VMs

To allow active resizing, install the spice agent

```
sudo emerge --ask app-emulation/spice-vdagent
```

This requires some kernel config. 

### Plymouth 

Install the plymouth package: 

```
sudo emerge --ask sys-boot/plymouth
sudo emerge --ask sys-boot/plymouth-openrc-plugin
```

Add this line to the `/etc/rc.conf`

```
rc_interactive="NO" 
```

Add the grub config lines: 

```
GRUB_CMDLINE_LINUX_DEFAULT='quiet splash'
GRUB_GFXMODE=1920x1080
#GRUB_GFXPAYLOAD_LINUX=keep
```

After installing Plymouth, the initramfs will need to be rebuilt. Dracut is able to do this. Simply rebuild the current initramfs. 

```
sudo dracut --hostonly --force
```

Finally, rebuild the grub config with these new params: 

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```





## Expand virual disk 

### Host OS system

Resize the qcow2 image with `qemu-img`, in this case growing by 20 gigs: 

```
sudo qemu-img resize gentoo.qcow2 +20G
```

### Guest OS system

Grow the partition with `parted`

```
sudo parted
> resize 3 100%
```

Expand the LVM Physical Volume

```
sudo pvresize /dev/sda3
```

Check that the pv was resized successfully

```
sudo vgdisplay stor
```



Extend the LVM object

```
sudo lvextend -l +100%Free /dev/stor/root
```

Finally, extend the underlying `ext4` filesystem

```
sudo resize2fs /dev/stor/root 
```



## Extras

### Utilities

```
sudo emerge -a app-i18n/unicode-emoji

sudo emerge -a kde-apps/okular
```

Crypto utils

```
sudo emerge -a app-crypt/keybase
sudo emerge -a app-crypt/kbfs

sudo emerge -a app-crypt/yubikey-manager app-crypt/yubikey-manager-qt
```

Shell enhancements

```
sudo emerge -a app-shells/bash-completion
```



### Filesystem Utils

```
sudo emerge -a sys-fs/fuse-common sys-fs/xfsprogs sys-fs/squashfs-tools sys-fs/exfat-utils sys-fs/fuse-exfat sys-fs/fuseiso sys-fs/ntfs3g
```





## Network Management 

```
sudo emerge -a net-misc/networkmanager  kde-misc/plasma-applet-network-monitor
```

Recompile any packages with changed use flags. 

```
euse -E networkmanager
```

Disable any network services

```
sudo rc-update del net.ens3
sudo rc-update del dhcpd
```

Enable NetworkManager

```
rc-service NetworkManager start
rc-update add NetworkManager default
```

### XFCE

```
net-libs/libsoup
```

```
emerge --ask xfce-base/xfce4-meta xfce-extra/xfce4-notifyd
```





# Unbreaking things

```
emerge --changed-use --deep @world	

```

