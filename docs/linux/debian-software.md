# Debian Software Updates

These are a couple useful tricks for updating a Debain (or derivative) system's packages, firmware, software, and more. 

## Firmware Updates

Fwupd is a neat tool to help manage device firmware. It usually comes installed by default if you select a desktop environment from tasksel. 

First, check if you have devices that support fwupd: 

    fwupdmgr get-devices

Check for updates: 

    fwupdmgr refresh
    fwupdmgr get-updates

Install updates: 

    fwupdmgr update

If there are no devices on your system that have supported firmware, the package can be removed safely. 

## Update Tools

### Changelogs

Useful for changelogs and finding missing programs/libraries: 

    sudo apt install apt-listchanges

This gives a more useful changelog view when updating: 

```
linux (5.10.113-1) bullseye-security; urgency=high

  * New upstream stable update:
    https://www.kernel.org/pub/linux/kernel/v5.x/ChangeLog-5.10.107
    - Revert "xfrm: state and policy should fail if XFRMA_IF_ID 0"
      (Closes: #1008299)
    - xfrm: Check if_id in xfrm_migrate
    - xfrm: Fix xfrm migrate issues when address family changes
    ...
```

This will also be mailed to root after a software update. 

I wouldn't recommend setting this up on a cluster of several servers, as you will get annoying duplicate emails. This is best used on 'pet' machines like personal laptops or home servers. 

### Bug reports

Useful for checking critical bugs on the Debian tracking system before installation. It is helpful to check if an update is safe before executing the package updates. 

    sudo apt install apt-listbugs reportbug

Example: 

```
$ apt-listbugs list thunderbird

Retrieving bug reports... Done
Parsing Found/Fixed information... Done
grave bugs of thunderbird (→ ) <Forwarded>
 b1 - #1014745 - thunderbird: Using "Reply All" is sometimes removing all Cc recipients
Summary:
 thunderbird(1 bug)
```

This will automatically kick in when performing an `apt upgrade` operation. 

### Apt-File

Install it using apt:

    sudo apt install -y apt-file

Once installed, update the index: 

    sudo apt-file update

`apt-file` helps find appropriate packages for a given file name. For example: 

```
$ apt-file search /bin/netstat
net-tools: /bin/netstat                   
netstat-nat: /usr/bin/netstat-nat
```

## Flatpak

Install and setup flatpak: 

    sudo apt install flatpak
    sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Show enabled repos: 

    flatpak remotes

Find software: 

    flatpak search foobar

Install software: 

    sudo flatpak install foobar

## Software Updates

### Checking for updates

    sudo apt update
    apt list --upgradable

Dry run mode

    apt upgrade -s > update_log.txt

### Apply all updates

    sudo apt upgrade -y

### Checking for pending restart

List the services that need 

    sudo needrestart -q -r l

### Cleanup system

Remove unneeded packages

    sudo apt autoremove
    sudo apt autoclean
    sudo apt clean

### Cleanup install

Cleanup an install to remove un-needed packages:

When using tasksel, lots of "baggage" will come along. I like to clean it up a little. 

    sudo apt remove gnome-games gnome-software gnome-software-common
