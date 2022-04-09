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


## PulseAudio power management 

Pulse sometimes hogs CPU, which can significantly shorten battery life. 

Add to `/etc/pulse/default.pa`

```
load-module module-udev-detect tsched = 0
```

See: [https://wiki.debian.org/PulseAudio#Excessive_CPU_usage_and_distortion](https://wiki.debian.org/PulseAudio#Excessive_CPU_usage_and_distortion)