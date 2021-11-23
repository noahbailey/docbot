# Syslog Alerts

`rsyslog` is default on most Debain based distros. 


The alert format should resemble this: 

```
Alert for %hostname%: 
Timestamp: %timereported%
Severity:  %syslogseverity-text%
Program:   %programname%
Message:   %msg%
```

When processed, this will create an email like this: 

```
Alert for foobar.local:
Timestamp: Nov 22 22:11:52
Severity:  Error
Program:   baz
Message:   Foobar is fubar'd!
```

## Configure rsyslog

Add this config to a new file,  `/etc/rsyslog.d/alert.conf`

```sh
$ModLoad ommail
$ActionMailSMTPServer 127.0.0.1
$ActionMailFrom rsyslog@localhost
$ActionMailTo root@localhost
$template mailSubject,"[%hostname%] Syslog alert for %programname%"
$template mailBody,"Alert for %hostname%:\n\nTimestamp: %timereported%\nSeverity:  %syslogseverity-text%\nProgram:   %programname%\nMessage:  %msg%"
$ActionMailSubject mailSubject
# make sure we receive an email only once per hour
$ActionExecOnlyOnceEveryInterval 3600
# capture all errors, excluding annoying OpenVPN auth message
if $syslogseverity <= 3 and not ($msg contains 'cannot locate HMAC') then :ommail:;mailBody
```

This will configure the daemon to send alerts on any message with the severity "Error" or higher. 

## Test the config

Rsyslog can self-test its config: 

    sudo rsyslogd -N1

## Apply the config

Reload the service to apply the changes: 

    sudo systemctl reload rsyslog

## Test alerts

Use the logger to test the config: 

    logger -p "local0.error" "This is a drill!" -t "servicename"


## More info

Template variables: [https://www.rsyslog.com/doc/master/configuration/properties.html](https://www.rsyslog.com/doc/master/configuration/properties.html)

Mail module: [https://www.rsyslog.com/doc/v8-stable/configuration/modules/ommail.html](https://www.rsyslog.com/doc/v8-stable/configuration/modules/ommail.html)
