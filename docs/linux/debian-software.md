# Debian Software Updates

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


## Checking for updates

    sudo apt update
    apt list --upgradable

Dry run mode

    apt upgrade -s > update_log.txt

## Apply all updates

    sudo apt upgrade -y

## Checking for pending restart

List the services that need 

    sudo needrestart -q -r l

## Cleanup debian

Remove unneeded packages

    sudo apt autoremove
    sudo apt autoclean
    sudo apt clean

Cleanup an install to remove un-needed packages:

When using tasksel, lots of "baggage" will come along. I like to clean it up a little. 

    sudo apt remove gnome-games gnome-software gnome-software-common
