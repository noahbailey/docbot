# FreeBSD

## Updating

Update base system: 

    freebsd-update fetch

    freebsd-update install

Update packages: 

    pkg update

    pkg upgrade

Cleanup old package versions: 

    pkg clean

## User shell

Change the shell to bash: 

    chsh -s /usr/local/bin/bash noah

## doas/sudo

Install doas

    pkg install doas

Configure doas by creating a file at `/usr/local/etc/doas.conf`

    permit persist keepenv :wheel

Optional: add an alias to sudo in the user's .bashrc
    
    alias sudo="doas"

# GUI system

## X11 

* https://docs.freebsd.org/en/books/handbook/x11/

Install xorg

    pkg install xorg xdm

Enable XDM by editing the `/etc/ttys` and changing the `off` to `on` under `ttyv8`

Add to `/etc/rc.conf`

    xdm_enable="YES"

Start the XDM service: 

    service xdm start

Install XFCE4 desktop env: 

    pkg install xfce

Enable Dbus on boot by adding this line to `/etc/rc.conf`

    dbus_enable="YES

Start xfce4 on login: 

    exec /usr/local/bin/startxfce4


## Applications

Install firefox: 

    sudo pkg install firefox

Install thunderbird: 

    sudo pkg install thunderbird


## Utilities

    sudo pkg install vim tmux htop

`dig` command can be installed with `bind-tools` 

    sudo pkg install bind-tools

Add dev and admin tools: 

    sudo pkg install gnupg git 
