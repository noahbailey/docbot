# Sysstat 

Sysstat is a very basic and light, but extremely powerful metrics system for Unix. 

## Install

Can be installed on most distributions. 

    sudo apt install sysstat 
    sudo pacman -S sysstat

## Enable Collection

Statistics collection is not enabled by default on some distros, typically Debian & Ubuntu derivitaves. 

Edit `/etc/default/sysstat`

    ENABLED="true"

The service can be enabled and restarted. 

    sudo systemctl enable sysstat
    sudo systemctl restart sysstat

## Stats Collection

On most distros, a cron job will be automatically created - On others it must be created manually. 

### Cron-style

`/etc/cron.d/sysstat`

    */2 * * * *   root  /usr/lib/sysstat/sa1 1  1
    59 23 * * *   root  /usr/lib/sysstat/sa1 60 2

Or if full stats collection is required: 

    */2 * * * *   root  /usr/lib/sysstat/sa1 1  1 -S ALL
    59 23 * * *   root  /usr/lib/sysstat/sa1 60 2 -A ALL

This job will log the statistics at a given time and save the output to `/var/log/saxx`

### Systemd Timer

The timer lives in `/lib/systemd/system/sysstat-collect.timer`. It is possible to edit the timer to lower the sample interval: 

```ini
[Unit]
Description=Run system activity accounting tool every 10 minutes

[Timer]
OnCalendar=*:00/2

[Install]
WantedBy=sysstat.service
```

Then, the service unit file can be optionally modified to collect power state: 

```ini
[Unit]
Description=system activity accounting tool
Documentation=man:sa1(8)
After=sysstat.service

[Service]
Type=oneshot
User=root
ExecStart=/usr/lib/sa/sa1 1 1 -S ALL
```

Then, systemd can be reloaded and enabled. 

    sudo systemctl daemon-reload

    sudo systemctl enable sysstat-collect.timer
    sudo systemctl start  sysstat-collect.timer


## Statistics

As an admin, stats can be read from sysstat with the `sar` command. 

    sar 

To see all stats from today: 

    sar -A

To see stats form yesterday: 

    sar -1


`sadf` can also be used to create graphics: 

    sadf -g > output.svg


### Stats Types

CPU stats: 

    sar -p

Memory stats: 

    sar -r 

IO stats: 

    sar -b

Network stats: 

    sar -n DEV

Socket stats: 

    sar -n SOCK

Load stats: 

    sar -q LOAD
    sar -q PSI


### Graphs

Since the `sadf` command takes `sar` args, you can make neat graphs with multiple stat types in one. 
Example: 

1. Graph system load and IO activity: 

        sadf -g -- -q LOAD -b > output.svg

2. Graph memory and swap: 

        sadf -g -- -r -W > output.svg

3. Temp/Fan for laptop

        sadf -g -- -m ALL > output.svg

4. Block device IO (disk usage)

        sadf -g -- -d > output.svg

### Additional Graphics options

Additional opts can be specified in the `sadf` command: 

1. Show all metrics, one type per row: 

        sadf -g -O packed -- -A > output.svg

2. Show 24 hours of metrics starting from midnight: 

        sadf -g -O oneday -T > output.svg


