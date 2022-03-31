# Plymouth for Debian

## Install

Generally you must have all of the correct drivers. 

    sudo apt install firmware-linux

Then, you can install plymouth: 

    sudo apt install plymouth plymouth-themes

## Config

Change `/etc/default/grub`

```ini
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_GFXMODE=1920x1080
```

Then update the grub config: 

    sudo update-grub2


### Set a theme

I like the 'spinner' theme as a simple and classy boot screen. 

    sudo plymouth-set-default-theme -R spinner

This will kick off an initramfs build. 

