# Arch: Installing PipeWire

## Install package

    sudo pacman -S pipewire


## Replace Pulse

    sudo pacman -S pipewire-pulse

```
    resolving dependencies...
    looking for conflicting packages...
    :: pipewire-pulse and pulseaudio are in conflict. Remove pulseaudio? [y/N] Y
    :: pipewire-pulse and pulseaudio-bluetooth are in conflict. Remove pulseaudio-bluetooth? [y/N] Y

    Packages (3) pulseaudio-14.0-1  pulseaudio-bluetooth-14.0-1  pipewire-pulse-0.3.17-1

    Total Download Size:  0.00 MiB
    Total Removed Size:   6.25 MiB

    :: Proceed with installation? [Y/n] 
```

## Activate user-level units

Start & enable both user-level services: 

    systemctl --user enable pipewire.service
    systemctl --user start pipewire.service

    systemctl --user enable pipwire-pulse.service
    systemctl --user start pipwire-pulse.service

