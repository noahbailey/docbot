# Monit 

* https://linux.die.net/man/1/monit
* https://mmonit.com/wiki/Monit/ConfigurationExamples

## Install

    sudo apt install monit

Optionally install postfix

    sudo apt install postfix mailutils

## Config

### Mail notifications

Set monit to send to root user: 

```
mailserver 127.0.0.1
set alert root@localhost
  but not on { instance }

set mail-format {
  from:    root@$HOST
  subject:  [$SERVICE] $DESCRIPTION
  message: Alert for $SERVICE
}
```

### Unix socket 

Control socket is used to check state

```
set httpd unixsocket /var/run/monit.sock
  allow user:pass
```

This allows status checks

    sudo monit status
    sudo monit summary

## Minimal Config

Example config file, needing only service checks. 

```
set daemon 60
set log /var/log/monit.log
set idfile /var/lib/monit/id
set statefile /var/lib/monit/state
set mailserver 127.0.0.1
set eventqueue
    basedir /var/lib/monit/events
    slots 250
set alert root@localhost
  but not on { instance }
set httpd unixsocket /var/run/monit.sock
  allow user:pass

set mail-format {
  from:    root@$HOST
  subject:  [$SERVICE] $DESCRIPTION
  message: Alert for $SERVICE
Date:        $DATE
Action:      $ACTION
Host:        $HOST
Description: $DESCRIPTION
}
```

# Alert example configurations


## System and Disk Check

Basic system params:

```
check system $HOST
  if loadavg (1min) per core > 2 for 5 cycles then alert
  if loadavg (5min) per core > 1.5 for 10 cycles then alert
  if cpu usage > 95% for 5 cycles then alert
  if memory usage > 90% then alert
  if swap usage > 50% then alert

check device root with path /
  if space usage > 90% then alert
  if inode usage > 90% then alert
  if changed fsflags then alert
  if service time > 10 milliseconds for 5 cycles then alert


```

## Network Interface check

Network status, and usage/throughput info: 

```
check network public with interface eth0
  if failed link then alert
  if changed link then alert
  if saturation > 90% then alert
  if download > 10 MB/s then alert
  if total uploaded > 1 GB in last hour then alert
```

## Network Reachability check

Test if the host is able to ping an internet host: 

```
check host REACHABILITY with address 1.1.1.1
  if failed ping with timeout 10 seconds then alert
```

## SSH server check

OpenSSH service status: 

```
check process sshd with pidfile /var/run/sshd.pid
  start program = "/usr/bin/systemctl start sshd"
  stop program  = "/usr/bin/systemctl stop sshd"
  if failed port 22 protocol ssh then restart
```

## Nginx check

Nginx status, including an HTTP probe: 

```
check process nginx with pidfile /var/run/nginx.pid
  start program = "/usr/bin/systemctl start nginx"
  stop program  = "/usr/bin/systemctl stop nginx"
  if failed port 80 protocol http for 2 cycles then restart
  if failed port 443 protocol https for 2 cycles then restart
```

## PHP-FPM check

Check if the PHP daemon is running, and the socket is functional: 

```
check process php-fpm with pidfile /var/run/php/php7.4-fpm.pid
  start program = "/usr/bin/systemctl start php7.4-fpm"
  stop program  = "/usr/bin/systemctl stop  php7.4-fpm"
  if failed unixsocket /var/run/php/php7.4-fpm.sock for 2 cycles then restart
  if cpu > 60% for 2 cycles then alert
  if cpu > 90% for 5 cycles then restart
  if memory usage > 1024 MB for 2 cycles then alert
  if memory usage > 8192 MB for 5 cycles then restart
```

## Mysql and Mariadb

Check if the process is running, and using a regular amount of system resources: 

```
check process mysqld with pidfile ls /var/run/mysqld/mysqld.pid
  start program = "/usr/bin/systemctl start mysqld"
  stop program  = "/usr/bin/systemctl stop  mysqld"
  if cpu > 60% for 2 cycles then alert
  if cpu > 90% for 5 cycles then restart
  if memory usage > 1024 MB for 2 cycles then alert
  if memory usage > 8192 MB for 5 cycles then restart
```


## DNS & DHCP server checks

Bind9 status: 

```
check process bind9 with pidfile /var/run/named/named.pid
  start program = "/usr/bin/systemctl start bind9"
  stop program  = "/usr/bin/systemctl stop  bind9"
  if failed host 127.0.0.1 
    port 53 type udp for 2 cycles then restart
```

DHCP server status: 

```
check process dhcpd with pidfile /var/run/dhcpd.pid
  start program = "/usr/bin/systemctl start isc-dhcp-server"
  stop program  = "/usr/bin/systemctl stop  isc-dhcp-server"
```

## Hardware checks

Install the sensor reading utilities: 

    sudo apt install smartmontools lm-sensors hddtemp

Then, create a cpu checker script at `/opt/proctemp.sh`

```sh
#!/bin/bash
TEMP=`tr -d '000' < /sys/class/thermal/thermal_zone0/temp`
exit $TEMP
```

And, a disk checker script at `/opt/disktemp.sh`

```sh
#!/bin/bash
TEMP=$(/usr/sbin/hddtemp -n /dev/sda)
exit $TEMP
```

Note that both scripts output the temperature measurement as the exit code. 

Then, the monit checks can read the code and decide how to handle the event based on the temperature measurements: 

```
check program PROCTEMP with path "/opt/proctemp.sh"
  every 5 cycles
  if status > 75 then alert

check program DISKTEMP with path "/opt/disktemp.sh"
  every 5 cycles
  if status > 60 then alert

check program DISKHEALTH with path "/usr/sbin/smartctl -H /dev/sda"
  every 5 cycles
  if status > 0 then alert
```
