# OpenSMTPD Mail Relay

SMTPD is derived from [OpenBSD](/bsd/openbsd/#local-opensmtpd-relay). 

## Install OpenSMTPD

Installation is a single package, with some prerequisites. 

    sudo apt install opensmtpd

## Configure OpenSMTPD 

`/etc/smtpd.conf`

```
listen on socket
listen on localhost

table aliases file:/etc/aliases
table secrets file:/etc/secrets

action "local" maildir alias <aliases>
action "relay" relay host smtp+tls://myrelay@smtp.mailgun.org:587 auth <secrets>

match for local action "local"
match for any action "relay"
```

`/etc/secrets`

```
myrelay webmaster@example.com:xxx-password-xxx
```

Set the permissions and ownership: 

    sudo chmod 640 /etc/secrets
    sudo chown root:opensmtpd /etc/secrets

Note: the manpages refer to the group `_smtpd`, but this group does not exist on Linux; only OpenBSD. 

## Aliases

Add to `/etc/aliases` to send root mail to your inbox: 

    root:   billy@example.com

Then, update the aliases db: 

    sudo newaliases

## Install a mailer

Any mailer can be used, this is the smallest one. You may want sendmail if you need to send file attachments. 

    sudo apt install bsd-mailx

## Send an email

If everything works correctly, you will be able to send emails to yourself: 

    echo "beep boop" | mail -s "Test email from your cool server!" root

That will send an email to the same address configured in the aliases file. 
