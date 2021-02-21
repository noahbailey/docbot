# OpenBSD

Sources: 

* https://sohcahtoa.org.uk/openbsd.html
* https://www.openbsd.org/faq/

## System Maintenance

The base system is maintained seperately from the userspace. 

### Updates/patches

    sudo syspatch


### Upgrades/releases

    sudo sysupgrade

### Software updates

    sudo pkg_add -u

### Verify integrity of packages

    sudo pkg_check

# Creature comforts

## Bash shell for user

    doas pkg_add bash

change the shell for yourself

    chsh -s /usr/local/bin/bash

Create a minimal `.bashrc`

    export PS1="[\u@\h \W] "

Create a minimal `.profile`

    if [ -s ~/.bashrc ]; then
      source ~/.bashrc;
    fi
 
## pseudo-sudo

`doas` is a less bad replacement for `sudo`

    cp /etc/examples/doas.conf /etc/doas.conf

Users in the `wheel` group are given sudo-ish priviledges by default. 

It can be made to act the same as sudo by creating some aliases in your .bashrc

    alias sudo="doas"

To make it more user-friendly, use the sudo-like behaviour of not asking for password after successful auth, edit `/etc/doas.conf`

    permit persist keepenv :wheel

### Aliases

`.bashrc` has lines added: 

    alias ll="ls -lh"
    
### useful CLI tools

    sudo pkg_add rsync vim htop neofetch

# GUI stuff (for humans only)

## Web browser

    pkg_add firefox-esr

## XFCE4 desktop environment

While not strictly required, extras does add useful components

    pkg_add xfce xfce-extras

Enable the services: 

    rcctl enable apmd
    rcctl set apmd flags -A
    rcctl start apmd

    rcctl enable messagebus
    rcctl start  messagebus


Edit the file `~/.xsession`

    exec /usr/local/bin/startxfce4

