# Systemd Units

The user, group, and service will be called `foobar`. 

## Service account

The unprivileged user will run the service. 

```sh
sudo addgroup --system foobar
sudo adduser --system \
    --disabled-login \
    --ingroup foobar \
    --shell /bin/nologin \
    --no-create-home foobar
```

## Systemd unit

The unit file should be created at `/etc/systemd/system/foobar.service`

```ini
[Unit]
Description=FooBar
Documentation=https://docbot.onetwoseven.one

[Service]
Type=simple
ExecStart=/opt/foobar/startup
ExecStop=/opt/foobar/stop
ExecReload=/opt/foobar/reload
PIDFile=/run/foobar.pid
WorkingDirectory=/opt/foobar/
User=foobar
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Once the unit file is created, it can be enabled and started: 

```sh
sudo systemctl daemon-reload
sudo systemctl start foobar.service
sudo systemctl enable foobar.service
```

### Environment variables

Variables can be specified in the unit file in the service section: 

```ini
[Service]
...
Environment=MYVAR="myvalue"
```

Or, an environment file can be created

```ini
MYVAR="myvalue"
```

And referenced in the unit file

```ini
EnvironmentFile=/opt/foobar/vars
```

## CGroups

You can enable cgroups for the unit to allow resource tracking and limits: 

```ini
[Service]
...
Slice=foobar.slice
MemoryAccounting=yes
CPUAccounting=yes
```

See also: [https://www.redhat.com/sysadmin/cgroups-part-four](https://www.redhat.com/sysadmin/cgroups-part-four)

## Sandboxing

Sandboxing can be enabled by turning on these options in the `[Service]` section: 

```ini
[Service]
...
ProtectSystem=full
ProtectHome=yes
PrivateDevices=yes
NoNewPrivileges=yes
PrivateTmp=yes
```

More extensive options, but more likely to cause issues. 

```ini
ProtectControlGroups=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectKernelLogs=yes
ProtectHostname=true
KeyringMode=private
MemoryDenyWriteExecute=yes
RestrictRealtime=yes
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM
```

Disallowing W^X frequently causes issues with Java applications. 

For more info: 

* [https://www.redhat.com/sysadmin/mastering-systemd](https://www.redhat.com/sysadmin/mastering-systemd)
* [https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Sandboxing](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Sandboxing)
