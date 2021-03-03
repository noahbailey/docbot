# Gentoo: WiFi

Reference: 

* https://packages.gentoo.org/packages/sys-firmware/iwl7260-ucode
* https://wiki.gentoo.org/wiki/Iwlwifi
* https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Final_Configuration_Steps#WiFi



### Linux firmware

```
echo sys-kernel/linux-firmware savedconfig >> /etc/portage/package.use/kernel

emerge --ask sys-kernel/linux-firmware
```



### Ucode

Setup package.accept_keywords

```
# Allow unstable branch for ucode
sys-firmware/iwl7260-ucode
sys-firmware/iwl3160-7260-bt-ucode
```

Then emerge the package: 

```
emerge -qa sys-firmware/iwl7260-ucode
```

