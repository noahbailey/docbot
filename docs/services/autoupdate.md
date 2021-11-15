# Auto-Update for Ubuntu/Debian

## Install

Unattended-upgrades is installed by default most of the time. If not, run this: 

    sudo apt install unattended-upgrades

    sudo systemctl enable --now unattended-upgrades

## Configure

The default config is mostly good enough. There are some times you may want to tweak it. 

The config file is located at `/etc/apt/apt.conf.d/50unattended-upgrades`

Minimum config: 

```
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
};
```

### Block updating a critical service

For example, prevent updates to Mysql or Postgres to avoid unplanned service restarts: 

```
Unattended-Upgrade::Package-Blacklist {
    "mysql-server";
    "postgresql-";
};
```

### Report failed updates 

This configuration will send a email report when an update has an error: 

```
Unattended-Upgrade::Mail "root@localhost";
Unattended-Upgrade::MailReport "only-on-error";
```

Note that this requires a local mail relay such as [postfix](/services/postfix) set up. 
