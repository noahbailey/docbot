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

## ZFS 

### List disks

Using geom: 

    geom disk list

Or, install `lsblk` as you would on Linux: 

    sudo pkg install lsblk
    lsblk

### Snapshots

Take a snapshot: 

    sudo zfs snapshot zroot@2021-06-30


List available snapshots: 

    zfs list -t snapshot

    NAME               USED  AVAIL     REFER  MOUNTPOINT
    zroot@2021-06-30     0B      -       96K  -


### Devices

List pools: 

    zpool list

    NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
    zroot  37.5G  3.23G  34.3G        -         -     1%     8%  1.00x    ONLINE  -

Check device health: 

    zpool status

    pool: zroot
    state: ONLINE
    config:

        NAME           STATE     READ WRITE CKSUM
        zroot          ONLINE       0     0     0
        vtbd0p3.eli  ONLINE       0     0     0

    errors: No known data errors

### Scrubs

Perform a disk scrub

    sudo zpool scrub zroot

Check scrub status: 

    zpool status 


    pool: zroot
    state: ONLINE
    scan: scrub in progress since Tue Jun 29 22:51:19 2021
        2.27G scanned at 232M/s, 260K issued at 26K/s, 3.23G total
        0B repaired, 0.01% done, no estimated completion time
    config:

        NAME           STATE     READ WRITE CKSUM
        zroot          ONLINE       0     0     0
        vtbd0p3.eli  ONLINE       0     0     0

    errors: No known data errors
