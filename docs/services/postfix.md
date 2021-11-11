# Postfix Mail Relay

Postfix is a flexible and powerful mail transfer agent service. Here, it will be used as a relay to send local messages to an external mailbox. 

## Install

Install the packages: 

    sudo apt install -y postfix mailutils libsasl2-2 libsasl2-modules ca-certificates ssl-cert

## Configure

Edit the file `/etc/postfix/main.cf`, replacing "mail.example.com" with the mail service you wish to relay email through. 

```
mynetworks = 127.0.0.0/8, [::1]/128
inet_interfaces = 127.0.0.1
relayhost = [mail.example.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
```

Then, add the credentials for the mail relay to `/etc/postfix/sasl_passwd`

```
[mail.example.com]:587 user@example.com:mysecurepass
```

After editing the file, update the hashed credentials:

    sudo postmap /etc/postfix/sasl_passwd

### Aliases

Aliases determine where mail will be routed for local users. 

Edit the `/etc/aliases` file and fill it in with your email addresses: 

```
postmaster: root
root: webmaster@example.com

#An example
ongo: ongo@gablogian.org
```

After editing the file, regenerate the aliases database: 

    sudo newaliases

## Send emails

Send an email to the root user: 

    echo "Hello, root!" | mail -s "A message for Charlie Root" root

A message should be delivered to your mailbox shortly. 
