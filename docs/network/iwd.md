# Replace wpa_supplicant with iwd

## Install iwd

    sudo apt install iwd

## Configure

Add to `/etc/NetworkManager/NetworkManager.conf`

```ini
[device]
wifi.backend=iwd
```

Stop all services:

    sudo systemctl stop NetworkManager.service
    sudo systemctl stop wpa_supplicant.service
    sudo systemctl disable wpa_supplicant.service

Configure iwd's config file: /etc/iwd/main.conf 

```ini
[Settings]
AutoConnect=true
```

Start services:

    sudo systemctl start iwd.service
    sudo systemctl start NetworkManager.service

After reconfiguring this, you will likely have to re-connect to any known wifi networks. I am not aware of a way to automatically merge configurations. 


## CLI

```
iwctl device list
                                    Devices                                    
--------------------------------------------------------------------------------
  Name                Address             Powered   Adapter   Mode      
--------------------------------------------------------------------------------
  wlan0               xx:xx:xx:xx:xx:xx   on        phy0      station   
```

