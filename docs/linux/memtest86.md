# Gentoo: Memtest

## Emerge the package

Emerge the sys-apps/memtest86+ package

```
sudo emerge --ask sys-apps/memtest86+
```

## Grub2 setup

Grub2 doesn't need any special config. Just rebuild the boot config

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## References

https://wiki.gentoo.org/wiki/Memtest86%2B