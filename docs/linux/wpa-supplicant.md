# WPA Supplicant

### Create WPA passphrase

```
wpa_passphrase "8Hz WAN IP" >> /etc/wpa_supplicant/wpa_supplicant.conf
```

### Connect to the network

```
/etc/init.d/wpa_supplicant start
```

### Set the clock

```
ntpdate 0.pool.ntp.org
```



### Startup SSHD

```
/etc/init.d/sshd start
mkdir -p .ssh
curl https://github.com/noahbailey.keys >> .ssh/authorized_keys
```
