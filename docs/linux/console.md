# Console port

For a server with a physical console port, or "COM" onboard, the kernel and bootloader can be configured to allow management over a serial connection. 

## Find the console port

Check the output of `/proc/tty/driver/serial`

```
serinfo:1.0 driver revision:
0: uart:16550A port:000003F8 irq:4 tx:35537 rx:204 RTS|DTR
1: uart:16550A port:000002F8 irq:3 tx:0 rx:0
2: uart:unknown port:000003E8 irq:4
3: uart:unknown port:000002E8 irq:3
```
This tell us to use `/dev/ttyS0`

## Configure GRUB

Set the console port to `/dev/ttyS0` in the GRUB bootloader. This also sets the baud rate to 115200.

```
GRUB_CMDLINE_LINUX='console=tty0 console=ttyS0,115200n8'
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```

Then, update the grub config:

```
sudo update-grub
```

## Enable getty

Enable the systemd unit for getty on the serial port:

```
systemctl enable serial-getty@ttyS0.service
```

## Reboot

Finally the box should be rebooted. The BIOS might also need to be configured for the correct baud rate on the builtin console port - it should match exactly the GRUB configuration. 

