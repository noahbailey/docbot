# Locate (File Indexer)

## Install

    sudo apt install plocate

Enable the timer for periodic file database updates

    sudo systemctl enable plocate-updatedb.timer
    sudo systemctl start  plocate-updatedb.timer

## Configure

Edit /etc/updatedb.conf to exclude directories:

```ini
PRUNEPATHS="... /backups"
```

## Use locate

Search your index for a filename:

    locate my-cool-file

Count the files with a given extension

    locate -c -r /etc/.*yml
