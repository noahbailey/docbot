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

## Replace a Failed Disk

Failed disk in array

```
          mirror-2               DEGRADED     0     0     0
            da2                  ONLINE       0     0     0
            6017198632328599818  FAULTED      0     0     0  was /dev/da1
            da3                  ONLINE       0     0     0
```

Find the ID of the failed disk

```
geom disk list | less
```

```
Geom name: da0
Providers:
1. Name: da0
   Mediasize: 2000398934016 (1.8T)
   ...
   descr: ATA WDC WD2002FFSX-6
   lunid: 50014ee2c137727d
   ident: WD-WCC6N1VSFLSC
   ...
```

Find this disk in the system. Yank it. Replace it. 

Repeat this to find the ID of the new disk. 

Replace the failed disk:

```
zpool replace tank 6017198632328599818 /dev/da0
```

Check the status of the resilver:

```
zpool status tank

 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Mar 17 19:04:42 2025
        283G scanned at 2.86G/s, 184G issued at 1.86G/s, 859G total
        0B resilvered, 21.39% done, 00:06:03 to go

        NAME                       STATE     READ WRITE CKSUM
        tank                       DEGRADED     0     0     0
          ...
          mirror-2                 DEGRADED     0     0     0
            da2                    ONLINE       0     0     0
            replacing-1            DEGRADED     0     0     0
              6017198632328599818  FAULTED      0     0     0  was /dev/da1
              da0                  ONLINE       0     0     0
            da3                    ONLINE       0     0     0
```

It will take a minute... Go for a walk. 


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
