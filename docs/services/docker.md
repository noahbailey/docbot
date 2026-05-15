# Docker

## Auto cleanup

If you don't regularly clean out the docker image cache, your disk will eventually fill up. There is no automatic garbage collection. 

I do this with a systemd timer, but it can also be done with a cron job if you're old fashioned. 

`/etc/systemd/system/docker-cleanup.service`

```
[Unit]
Description=Cleanup Docker Images

[Service]
Type=oneshot
ExecStart=/usr/bin/docker system prune -a -f
User=root
```

`/etc/systemd/system/docker-cleanup.timer`

```
[Unit]
Description=Cleanup Docker Images

[Timer]
OnCalendar=weekly

[Install]
WantedBy=timers.target
```

Activate the timer

    sudo systemctl enable --now docker-cleanup.timer

