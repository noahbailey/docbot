# ClamAV

Install an antivirus scanner for your web server. 

    sudo apt install clamav clamav-daemon clamdscan

After installing, the `clamav-freshclam` service will download initial definitions. After a minute or so you can startup the main service: 

    sudo systemctl start clamav-daemon.service

The main socket is located at `/var/run/clamav/clamd.ctl`

### Manual AV Scan

Invoke a manual scan: 

sudo clamdscan --fdpass --multiscan /var/www/nextcloud

This will take a very long time to run, and use absolutely all of your CPU resources. Be careful. 


## Automated web server scan

A script to periodically scan the data directory: 

`/opt/scan.sh`

```shell
#!/bin/bash
monit unmonitor "$(hostname)"
clamdscan --infected --fdpass --multiscan --no-summary --move=/var/cache/clamdjail /var/www/nextcloud/data/
monit monitor "$(hostname)"
```

Note that infected files are automatically moved to `/var/cache/clamdjail` for quarantine. This script also disables the monitoring alert for the host to prevent annoying "high CPU" alerts while the scan is underway. 

Example output: 

```
/var/www/server/upload/totally_legit_file: Win.Trojan.Agent-6566022-0 FOUND
/var/www/server/upload/totally_legit_file: moved to '/var/cache/clamdjail/kmspico.tar.gz.d1518823493'
```

A crontab to automate the scan script, for example, scan the whole system once per week on wednesday: 

`/etc/cron.d/clamdscan`

```
MAILTO=root
0 2 * * 3   root    /opt/scan.sh
```
