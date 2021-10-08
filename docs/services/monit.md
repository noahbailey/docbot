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

### Unix socket 

Control socket is used to check state

```
set httpd unixsocket /var/run/monit.sock
  allow user:pass
```

This allows status checks

    sudo monit status
    sudo monit summary

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
    if space usage > 90% then alert
    if inode usage > 90% then alert

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
  start program = "/usr/bin/systemctl start sshd"
  stop program  = "/usr/bin/systemctl stop sshd"
  if failed port 22 protocol ssh then restart
```

Nginx status: 

```
check process nginx with pidfile /var/run/nginx.pid
  start program = "/usr/bin/systemctl start nginx"
  stop program  = "/usr/bin/systemctl stop nginx"
  if failed port 80  for 2 cycles then restart
  if failed port 443 for 2 cycles then restart
```
