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
set mailserver localhost
set alert root@localhost

set mail-format {
    from:    Monit <monit@$HOST>
    subject: alert --  $EVENT $SERVICE
    message: $EVENT Service $SERVICE
                Timestamp:   $DATE
                Action:      $ACTION
                Description: $DESCRIPTION
}
```

### System alerts

Basic system params:

```
check system $HOST
  if loadavg (1min) per core > 2 for 5 cycles then alert
  if loadavg (5min) per core > 1.5 for 10 cycles then alert
  if cpu usage > 95% for 5 cycles then alert
  if memory usage > 90% then alert
  if swap usage > 50% then alert

check device root with path /
    if SPACE usage > 80% then alert

```

Network status: 

```
check network public with interface eth0
  if failed link then alert
  if changed link then alert
  if saturation > 90% then alert
  if download > 10 MB/s then alert
  if total uploaded > 1 GB in last hour then alert
```

OpenSSH service status: 

```
check process sshd with pidfile /var/run/sshd.pid
  start program  "/etc/init.d/sshd start"
  stop program  "/etc/init.d/sshd stop"
  if failed port 22 protocol ssh then restart
```

Nginx status: 

```
check process nginx with pidfile /var/run/nginx.pid
  start program = "/etc/init.d/nginx start"
  stop program  = "/etc/init.d/nginx stop"
  group www-data
```
