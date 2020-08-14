# MySQL Server

There are two completely different things known as 'mysql': 

* The evil one maintaine by Oracle
* The less evil one known as 'MariaDB'

## Install MariaDB (Ideal)

```
sudo apt install mariadb-server mariadb-client
```

## Install MySQL (Not ideal)

```
sudo apt install mysql-server mysql-client
```


## Create Database and User

```SQL
CREATE DATABASE app-db;
CREATE USER 'app-user' IDENTIFIED BY 'xxxxxxxxxxxxxxxxxx';
GRANT ALL PRIVILEGES ON app-db.* TO 'app-user';
```

## MySQL Backup script

Create directory: 

    sudo mkdir -p /var/lib/mysql-backups

Write a script - `/opt/scripts/backups/backup-db.sh`

```bash
#!/bin/bash
set -eo pipefail
NOW=$(date +"%Y-%m-%d-%H-%M-%S")
mysqldump my-database | gzip > /var/lib/mysql-backups/my-database-$NOW.sql.gz
find /var/lib/mysql-backups/ -mtime +7 -delete
```

Create the cron job -> `/etc/cron.d/mysql-backup`

```
MAILTO="root"
00 */6 * * *    root    /opt/scripts/backup/backup-db.sh
```
