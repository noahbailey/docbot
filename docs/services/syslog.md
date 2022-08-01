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

This will configure the daemon to send alerts on any message with the severity "Error" or higher. 

The config snippet below will be added to a new config file, such as  `/etc/rsyslog.d/alert.conf`


```sh
module(load="ommail")
template (name="mailBody"  type="string" string="Alert for %hostname%:\n\nTimestamp: %timereported%\nSeverity:  %syslogseverity-text%\nProgram:   %programname%\nMessage:  %msg%")
template (name="mailSubject" type="string" string="[%hostname%] Syslog alert for %programname%")

if $syslogseverity <= 3 then {
   action(type="ommail" server="127.0.0.1" port="25"
          mailfrom="rsyslog@localhost"
          mailto="root@localhost"
          subject.template="mailSubject"
          template="mailBody"
          action.execonlyonceeveryinterval="3600")
}
```

To exclude services from email alerts, modify the `if` line. For example, excluding spammy alerts from smartd: 

```
if $syslogseverity <= 3 and $programname != 'smartd' then {
```

## Test the rsyslog config

Rsyslog can self-test its config: 

    sudo rsyslogd -N1

## Apply the rsyslog config

Reload the service to apply the changes: 

    sudo systemctl restart rsyslog

## Test alerts

Use the logger to test the config: 

    logger -p "local0.error" "This is a drill!" -t "servicename"


## More info

Template variables: [https://www.rsyslog.com/doc/master/configuration/properties.html](https://www.rsyslog.com/doc/master/configuration/properties.html)

Mail module: [https://www.rsyslog.com/doc/v8-stable/configuration/modules/ommail.html](https://www.rsyslog.com/doc/v8-stable/configuration/modules/ommail.html)
