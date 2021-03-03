# Arch: Plasma Desktop

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


## File Manager

Install the default KDE file manager: 

    sudo pacman -S dolphin

## Power Management

    sudo pacman -S acpid powertop tlp powerdevil

Make sure the services are running: 

    systemctl status tlp

Reference: [https://wiki.archlinux.org/index.php/Power_management](https://wiki.archlinux.org/index.php/Power_management)

## Bluetooth

    sudo pacman -S bluez blues-utils bluedevil

Enable the bluetooth service: 

    sudo systemctl enable --now bluetooth.service

Reference: [https://wiki.archlinux.org/index.php/Bluetooth](https://wiki.archlinux.org/index.php/Bluetooth)

## Consistent theming

Consistent theming for Plasma with GTK applications. 

    sudo pacman -S breeze-gtk kde-gtk-config

* https://wiki.archlinux.org/index.php/Uniform_look_for_Qt_and_GTK_applications
* https://wiki.archlinux.org/index.php/KDE#Plasma

