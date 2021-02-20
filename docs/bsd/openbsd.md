# OpenBSD

## System Maintenance

### Updates/patches

    sudo syspatch


### Upgrades/releases

    sudo sysupgrade

### Software updates

    sudo pkg_add -u

### Verify integrity of packages

    sudo pkg_check

## Creature comforts

## Web browser

    pkg_add firefox-esr

## XFCE4 desktop environment

    pkg_add xfce

Edit the file `~/.xsession`

    exec /usr/local/bin/startxfce4

## pseudo-sudo

`doas` is a less bad replacement for `sudo`

    cp /etc/examples/doas.conf /etc/doas.conf

Users in the `wheel` group are given sudo-ish priviledges by default. 

It can be made to act the same as sudo by creating some aliases in your .bashrc

    alias sudo="doas"

 
