# DD (Disk Destroyer)

## Burn an ISO to a flash drive

    sudo dd bs=4M if=image.iso of=/dev/sdx conv=fsync status=progress

## Secure-ish wipe of a drive

    sudo dd bs=4M if=/dev/urandom of=/dev/sdx conv=fsync 
