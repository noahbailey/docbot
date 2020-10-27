# Shrink an LVM partition

To shrink both the LVM volume _and_ the ext4 filesystem at the same time, first check: 

* The volume you want to reduce is unmounted
* There is free space in the filesystem

## Shrink/resize

For example, to reduce the size of the `/home` volume to 100G

    lvresize --resizefs --size 100G /dev/stor/home

