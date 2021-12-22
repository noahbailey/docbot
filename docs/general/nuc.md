# NUC on Linux

## NIC driver 

NUC6CAYH has issues with the regular kernel driver. Install the DKMS driver for the NIC: 

    sudo apt install r8168-dkms


```
Backing up initrd.img-5.10.0-9-amd64 to /boot/initrd.img-5.10.0-9-amd64.old-dkms
Making new initrd.img-5.10.0-9-amd64
(If next boot fails, revert to initrd.img-5.10.0-9-amd64.old-dkms image)
update-initramfs.........
```

## Microcode updates

Make sure you have non-free sources enabled first. 

```
deb http://deb.debian.org/debian bullseye main contrib non-free
deb http://security.debian.org/debian-security bullseye-security main contrib non-free
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
```

Then install the microcode update package: 

    apt install intel-microcode

Then, reboot the host. 


## Integrated LED control

In the BIOS, all LEDs must be set to "SW Control". 

Then, the kernel driver can be installed. 

### Acpi kernel driver

    git clone https://github.com/milesp20/intel_nuc_led.git
    cd intel_nuc_led

Build the dpkg to install: 

    sudo make dkms-deb

Then install the generated package: 

    sudo dpkg -i /var/lib/dkms/intel-nuc-led/1.0/deb/intel-nuc-led-dkms_1.0_all.deb

The module will need to be enabled: 

    sudo modprobe nuc_led

It should also be loaded at bootup time: 

    echo "nuc_led" | sudo tee -a /etc/modules-load.d/nuc.conf

### LED Control

The LEDs can be controlled by the device file `/proc/acpi/nuc_led`

```
cat /proc/acpi/nuc_led 

Power LED Brightness: 100%
Power LED Blink/Fade: Always On (0x04)
Power LED Color: Blue (0x01)

Ring LED Brightness: 20%
Ring LED Blink/Fade: Always On (0x04)
Ring LED Color: Green (0x06)
```

It can be changed by sending data into that pseudo file, for example setting the ring to Green: 

    echo 'ring,20,none,blue' | sudo tee /proc/acpi/nuc_led


This can be used in scripts to indicate server problems visually: 

`/opt/nuc/ok.sh`

```sh
#!/bin/sh
echo 'ring,20,none,green' > /proc/acpi/nuc_led
```

`/opt/nuc/warn.sh`

```sh
#!/bin/sh
echo 'ring,20,none,yellow' > /proc/acpi/nuc_led
```

`/opt/nuc/alert.sh`

```sh
#!/bin/sh
echo 'ring,50,none,red' > /proc/acpi/nuc_led
```

`/opt/nuc/critical.sh`

```sh
#!/bin/sh
echo 'ring,50,blink_medium,red' > /proc/acpi/nuc_led
```

Example: if a backup job fails, flag a warning on the system: 

```
0 0 * * *   root   /opt/scripts/backup.sh || /opt/nuc/warn.sh
```
