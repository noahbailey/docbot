# Debian Software Updates

## Cleanup debian

When using tasksel, lots of "baggage" will come along. I like to clean it up a little. 

    sudo apt remove gnome-games gnome-software gnome-software-common


## Fwupd

Fwupd is a neat tool to help manage device firmware. It usually comes installed by default if you select a desktop environment from tasksel. 

First, check if you have devices that support fwupd: 

    fwupdmgr get-devices

Check for updates: 

    fwupdmgr refresh
    fwupdmgr get-updates

Install updates: 

    fwupdmgr update

If there are no devices on your system that have supported firmware, the package can be removed safely. 

