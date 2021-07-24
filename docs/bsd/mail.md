# Mail relay for FreeBSD

## Disable Sendmail

Stop the services in `/etc/rc.conf`

```
sendmail_enable="NO"
sendmail_submit_enable="NO"
sendmail_outbound_enable="NO"
sendmail_msp_queue_enable="NO"
```

## Stop Sendmail

Stop the service

    service sendmail stop

## Install OpenSMTPD

Install the package from ports

    pkg install opensmtpd

### Configure SMTPD

Add the relay credentials to `/etc/mail/secrets`

    myrelay webmaster@example.com:xxx-password-xxx


`/usr/local/etc/mail/smtpd.conf`

    listen on socket
    listen on lo0

    table aliases file:/etc/mail/aliases
    table secrets file:/etc/mail/secrets

    action "local" mbox alias <aliases>
    action "relay" relay host smtp+tls://myrelay@smtp.mailgun.org:587 auth <secrets>

    match for local action "local"
    match for any action "relay"

Enable SMTPD in `/etc/rc.conf`

    smtpd_enable="YES"

Edit `/etc/mail/aliases`

    root: root@onetwoseven.one

And run: 

    newaliases

Configure the system to use smtpd in `/etc/mail/mailer.conf`

```
sendmail        /usr/local/sbin/smtpctl
send-mail       /usr/local/sbin/smtpctl
mailq           /usr/local/sbin/smtpctl
makemap         /usr/local/sbin/smtpctl
newaliases      /usr/local/sbin/smtpctl
hoststat        /usr/bin/true
purgestat       /usr/bin/true

```

Add a symlink: 


    ln -sf /usr/local/sbin/smtpctl /usr/local/sbin/sendmail

