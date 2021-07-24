# ZFS on FreeBSD

## List disks

Using geom: 

    geom disk list

Or, install `lsblk` as you would on Linux: 

    sudo pkg install lsblk
    lsblk

## Create pool

Create a basic mirror pool: 

    zpool create tank mirror /dev/da0 /dev/da1

Create a striped-mirror pool (similar to raid-10): 

    zpool create tank mirror /dev/da0 /dev/da1 mirror /dev/da2 /dev/da3

## Create dataset

/tank/backups
    
    zfs create -o compress=lz4 tank/backups

## Snapshots

Take a snapshot: 

    sudo zfs snapshot -r zroot@now

Take a snapshot, with an automatically generated name: 

    sudo zfs snapshot -r zroot@$(date "+%Y%m%d%H%M.%S")


List available snapshots: 

    zfs list -t snapshot

    NAME               USED  AVAIL     REFER  MOUNTPOINT
    zroot@2021-06-30     0B      -       96K  -

Delete a snapshot: 

    sudo zfs destroy zroot@2021-06-30


### Compare a snapshot state: 

```
zfs diff zroot/usr/home@202107011206.09
Password:
M	/usr/home/noah/.bash_history
+	/usr/home/noah/Desktop/1.txt
+	/usr/home/noah/Desktop/2.txt
+	/usr/home/noah/Desktop/3.txt
+	/usr/home/noah/Desktop/4.txt
M	/usr/home/noah/Desktop
```

## Devices

List pools: 

    zpool list

    NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
    zroot  37.5G  3.23G  34.3G        -         -     1%     8%  1.00x    ONLINE  -

Check device health: 

    zpool status

    pool: zroot
    state: ONLINE
    config:

        NAME           STATE     READ WRITE CKSUM
        zroot          ONLINE       0     0     0
        vtbd0p3.eli  ONLINE       0     0     0

    errors: No known data errors

## Scrubs

Perform a disk scrub

    sudo zpool scrub zroot

Check scrub status: 

    zpool status 


    pool: zroot
    state: ONLINE
    scan: scrub in progress since Tue Jun 29 22:51:19 2021
        2.27G scanned at 232M/s, 260K issued at 26K/s, 3.23G total
        0B repaired, 0.01% done, no estimated completion time
    config:

        NAME           STATE     READ WRITE CKSUM
        zroot          ONLINE       0     0     0
        vtbd0p3.eli  ONLINE       0     0     0

    errors: No known data errors


## Replication

Send a snapshot on the local system, to another array on the local system

    zfs send -R -v zroot@202107011210.16 | zfs recv tank/zroot-backup

Send a differential backup based on yesterday's snapshot: 

    zfs send -R -v -i zroot@202107011210.16 zroot@202107022003.05 | zfs recv tank/zroot-backup


### Automatic snapshot script

```sh
#!/bin/sh

NUM_SNAPSHOTS=`zfs list -t snapshot zroot | awk '/@/{print $1}' | wc -l`
echo "[+] ${NUM_SNAPSHOTS} snapshots on volume"

echo "[+] Getting last snapshot on system"
LAST_SNAPSHOT=`zfs list -t snapshot zroot | tail -n1 | awk '{print $1}'`

echo "[+] Taking system snapshot"
zfs snapshot -r zroot@$(date "+%Y%m%d-%H%M")
NEW_SNAPSHOT=`zfs list -t snapshot zroot | tail -n1 | awk '{print $1}'`

echo "[+] Sending incremental backup"
zfs send -R -v -i $LAST_SNAPSHOT $NEW_SNAPSHOT | zfs recv tank/zroot-backup

echo "[+] Deleting oldest snapshot"
OLDEST_SNAPSHOT=`zfs list -t snapshot zroot | awk '/@/{print $1}' | head -n1`
zfs destroy -r -v "${OLDEST_SNAPSHOT}"
echo "[+] Deleted snapshot $OLDEST_SNAPSHOT"
```
