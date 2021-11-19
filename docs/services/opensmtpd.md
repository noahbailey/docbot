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

accept for local alias <aliases> deliver to mbox
accept for any relay via tls+auth://myrelay@smtp.mailgun.org:587 auth <secrets>
```

`/etc/secrets`

```
myrelay webmaster@example.com:xxx-password-xxx
```

Set the permissions and ownership: 

    sudo chmod 640 /etc/secrets
    sudo chown root:opensmtpd /etc/secrets

Note: the manpages refer to the group `_smtpd`, but this group does not exist on Linux; only OpenBSD. 

## Enable and start the service

After editing the config files, restart the service: 

    sudo systemctl restart opensmtpd

Make sure it is enabled: 

    sudo systemctl enable opensmtpd

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

## Check the firewall

If your system has a firewall configured, make sure there is a rule to allow local programs connect to tcp/25 for smtp: 

    sudo iptables -A INPUT -s 127.0.0.1 -p tcp -m tcp --dport 25 -j ACCEPT
