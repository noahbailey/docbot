# ZFS on FreeBSD

## List disks

Using geom: 

    geom disk list

Or, install `lsblk` as you would on Linux: 

    sudo pkg install lsblk
    lsblk

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
