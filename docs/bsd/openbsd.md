# OpenBSD

Sources: 

* https://sohcahtoa.org.uk/openbsd.html
* https://www.openbsd.org/faq/

# Installation

* https://www.openbsd.org/faq/faq14.html#softraid

To install with full disk encryption, exit to the shell in the installer menu (s). 

First, create the device node for the primary SATA disk `sd0`.  

    cd /dev && sh MAKEDEV sd0

For MBR boot, create a partition table: 

    fidsk -iy sd0

Then create the partition mapping for the softraid volume: 

    disklabel -E sd0
        a a
        [64]
        * 
        RAID
        w
        q

Create the encrypted device on the 'a' partition: 

    bioctl -c C -l sd0a softraid0

This will create a new 'cryto volume'. If booted from Cd, this will be `sd1`, but if using USB to install the OS this will be `sd2`. 

    cd /dev && sh MAKEDEV sd2

Finally, the first 1M will be overwritten to make sure there's no tainted data in the MBR blocks. 

    dd if=/dev/zero of=/dev/rsdc2c bs=1m count=1

Then, exit to the main installer and continue the process as usual. When asked for a root device point it towards the crypto volume created earlier, `sd2` in this example. 

## System Maintenance

The base system is maintained seperately from the userspace. 

### Updates/patches

To apply patches for the current version:

    # syspatch

Or, to upgrade the system to a new release: 

    # sysupgrade

### Software updates

Update all packages (ports tree)

    # pkg_add -u -iv

### Verify integrity of packages

    # pkg_check

### Clean up packages

This would be similar to `apt autoremove` on debian derivatives

    # pkg_delete -a -iv

# Creature comforts

## Bash shell for user

    # pkg_add bash

change the shell for yourself

    chsh -s /usr/local/bin/bash

Create a minimal `.bashrc`

    export PS1="[\u@\h \W] "

Create a minimal `.profile`

    if [ -s ~/.bashrc ]; then
      source ~/.bashrc;
    fi
 
## pseudo-sudo

`doas` is a less bad replacement for `sudo`

    cp /etc/examples/doas.conf /etc/doas.conf

Users in the `wheel` group are given sudo-ish priviledges by default. 

It can be made to act the same as sudo by creating some aliases in your .bashrc

    alias sudo="doas"

To make it more user-friendly, use the sudo-like behaviour of not asking for password after successful auth, edit `/etc/doas.conf`

    permit persist keepenv :wheel

### Aliases

`.bashrc` has lines added: 

    alias ll="ls -lh"
    
### useful CLI tools

    sudo pkg_add rsync vim htop neofetch

## Mail Relay

`/etc/mail/secrets`

    myrelay webmaster@example.com:xxx-password-xxx

Set the ownership and permissions: 

    # chown root:_smtpd /etc/mail/secrets
    # chmod 0640 /etc/mail/secrets

Configure the local smtpd daemon to send all matching messages through the relay server, in this case using the mailgun service: 

```
listen on socket
listen on lo0

table aliases file:/etc/mail/aliases
table secrets file:/etc/mail/secrets

action "local" mbox alias <aliases>
action "relay" relay host smtp+tls://myrelay@smtp.mailgun.org:587 auth <secrets>

match for local action "local"
match for any action "relay"
```

After changing the config file, restart the service: 
    
    # rcctl restart smtpd

Then, add your email addresses to `/etc/mail/aliases` to allow their mail to be sent to the relay:

    root: root@example.com
    bill: bill@example.com

After editing the aliases file, reload the aliases db: 

    # newaliases

Test the config by sending yourself an email: 

    echo "test" | mail -s "test email" jeb.kerman@example.com

# GUI stuff (for humans only)

## Web browser

    pkg_add firefox-esr

## XFCE4 desktop environment

While not strictly required, extras does add useful components

    pkg_add xfce xfce-extras

Enable the services: 

    rcctl enable apmd
    rcctl set apmd flags -A
    rcctl start apmd

    rcctl enable messagebus
    rcctl start  messagebus


Edit the file `~/.xsession`

    exec /usr/local/bin/startxfce4

Add your user to the system groups: 

    usermod -G staff billy
    usermod -G operator billy

## Trackpoint settings for Thinkpad

Add to the `~/.xsession` at the very top of the file (ie. before it starts X11)

```
# Set trackpoint acell
xinput set-prop /dev/wsmouse "Device Accel Constant Deceleration" 0.4

# Set up middle-click scroll
xinput set-prop "/dev/wsmouse" "WS Pointer Wheel Emulation" 1
xinput set-prop "/dev/wsmouse" "WS Pointer Wheel Emulation Button" 2
xinput set-prop "/dev/wsmouse" "WS Pointer Wheel Emulation Axes" 6 7 4 5
```


