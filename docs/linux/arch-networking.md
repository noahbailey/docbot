---
title: "Arch: Networking"
description: 
published: true
date: 2019-07-20T19:49:16.327Z
tags: 
---

# Networking

## DHCP Client

Install the ISC-DHCP client package: 

    sudo pacman -S dhclient

## Wireless Configuration 

### Basic wireless

For basic wireless configuration, the package `wifi-menu` can be installed. It is extremely primitive and basic but better than absolutely no network capability. 

### Drivers

For Intel wireless devices, ex. 7260 etc. 

    sudo pacman -S firmware-iwlwifi


## Network Management

### NetworkManager

Install NetworkManager

    sudo pacman -S networkmanager networkmanager-openvpn network-manager-applet

Set the service to start automatically

    sudo systemctl enable NetworkManager
    sudo systemctl start NetworkManager

### Firewall

Install the "uncomplicated firewall"

    sudo pacman -S ufw

Configure it with "sane defaults" 

    sudo ufw default-deny 
    sudo ufw limit ssh
    sudo ufw enable

It will now automatically start on boot. 
