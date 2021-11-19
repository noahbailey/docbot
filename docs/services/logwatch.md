# Logwatch

Logwatch can be set up to send weekly server reports. 

First, set up a [local mail relay](/services/postfix) and aliases to send root mail to your mailbox. 

## Install

    sudo apt install logwatch

## Configure 

Set up weekly report

```
LogDir = /var/log
TmpDir = /var/cache/logwatch
Output = mail
Format = text
Encode = none
MailTo = root
MailFrom = Logwatch
Archives = Yes
Range = between -7 days and -1 days
Detail = Low
Service = All
Service = "-zz-network"
Service = "-zz-sys"
Service = "-eximstats"
mailer = "/usr/sbin/sendmail -t"
```

Then, delete the default daily script: 

    sudo rm -v /etc/cron.daily/00logwatch 

And create a weekly cron task

    sudo vim /etc/cron.d/logwatch

```
# Run weekly - Monday at midnight
MAILTO="root"
0 0 * * 1   root    /usr/sbin/logwatch
```
