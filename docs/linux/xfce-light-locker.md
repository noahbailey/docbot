# XFCE4 with Light-Locker

Sometimes the screen will flash the desktop from before the session was locked. Not only is this annoying, it's also a privacy risk. 

This can be fixed by instructing X11 to use a specific driver instead of guessing. 

`/etc/X11/xorg.conf.d/20-intel.conf`

```
Section "Device"
 Identifier  "Intel Graphics"
 Driver      "Intel"
EndSection
```
