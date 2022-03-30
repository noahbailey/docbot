# Power Management

For a laptop, it's very important to properly configure the power management components. 

## TLP

    sudo apt install tlp smartmontools acpi

Enable TLP

    sudo systemctl enable tlp
    sudo systemctl start tlp

## Battery care

Uncomment these lines in `/etc/tlp.conf`

```
START_CHARGE_THRESH_BAT0=75
STOP_CHARGE_THRESH_BAT0=80
```

The service needs a quick restart for it to take effect: 

    sudo systemctl restart tlp

See: [https://linrunner.de/tlp/settings/battery.html](https://linrunner.de/tlp/settings/battery.html)

