# MDNS (Bonjour) and Printing

MDNS is a nice to have feature for small local networks. 

## Install OpenMDNS

    pkg_add openmdns

## Enable MDNS

Edit /etc/rc.conf.local

```
multicast=YES
mdnsd_flags=iwn0
```

### Test MDNS

You can test the mdns service by enumerating the local network: 

    mdnsctl browse

Some familiar devices should appear in the list. 

## Install CUPS

Common Unix Printing System

    pkg_add cups

Enable the daemon

    rcctl enable cupsd
    rcctl start  cupsd

Browse to http://localhost:631/admin and add a new printer. The network printer should appear under 'add printer'

## Printing from GUI programs

Install the package. No further configuration is needed. 

    pkg_add gtk+3-cups

Apps such as web browsers and PDF readers should be able to print. 

