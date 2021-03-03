# Gentoo: Plasma Desktop



### Set the profile

Chose a new profile: 

```
sudo eselect profile set default/linux/amd64/17.1/desktop/plasma
```

And apply it

```
sudo emerge --update --deep --newuse -av @world 
```

May require dispatch-conf

If there are dependancy issues, try adding a package.use entry. Example: 

```
*/* -bluetooth
```

### Install the desktop environment

```
sudo emerge -a kde-plasma/plasma-meta
sudo emerge -a kde-apps/dolphin
```

Set `sddm` as the display manager - edit `/etc/conf.d/xdm` 

```
DISPLAYMANAGER="sddm"
```

Set the sddm service to start automatically

```
sudo rc-update add xdm default
```



### Terminals

```
sudo emerge --ask x11-terms/alacritty
```



### Web Browsers

Optional - Install Firefox 

```
sudo emerge --ask www-client/firefox
```

Optional - Install Chromium

```
sudo emerge --ask www-client/chromium
```

> Chromium is garbage and anybody that uses it should be sent to the glue factory. 

```
sudo emerge -a kde-plasma/plasma-browser-integration
```

