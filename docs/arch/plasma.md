---
title: Plasma Desktop
description: 
published: true
date: 2019-07-20T19:49:20.432Z
tags: 
---

# Installing Plasma Desktop

## Well-Known User Directories

Install the XDG utils package: 

    sudo pacman -S xdg-user-dirs

The home folder directories will be generated on new login, or if manually updated: 

    xdg-user-dirs-update



## Base Xorg system

Install the base display server system: 

    sudo pacman -S xf86-video-intel xorg-server xorg-apps 

## Desktop Environment

Install the KDE Plasma 5 packages: 

    sudo pacman -S plasma 

## Display Manager

Install SDDM: 

    sudo pacman -S sddm

Configure SDDM as the primary display manager on the system: 

    sudo systemctl enable sddm 
    sudo systemctl start sddm 

Check by viewing `display-manager` service; it should contain references to SDDM. 

#### /etc/systemd/system/display-manager.service

```ini
[Unit]
Description=Simple Desktop Display Manager
Documentation=man:sddm(1) man:sddm.conf(5)
Conflicts=getty@tty1.service
After=systemd-user-sessions.service getty@tty1.service plymouth-quit.service systemd-logind.service

[Service]
ExecStart=/usr/bin/sddm
Restart=always

[Install]
Alias=display-manager.service
```

## File Manager

Install the default KDE file manager: 

    sudo pacman -S dolphin

