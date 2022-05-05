# Debian Backports

Backports are extra software available on a "future" distro release. 

From the offial debian backports website: 

> You are running Debian stable, because you prefer the Debian stable tree. It runs great, there is just one problem: the software is a little bit outdated compared to other distributions. This is where backports come in.

To enable backports, you must add an additional software repository:

```
echo "deb http://deb.debian.org/debian bullseye-backports main" | sudo tee -a /etc/apt/sources.list.d/backports.list

sudo apt update
```

Then, to install software, you must specify that you want the version from backports: 

    sudo apt install -t bullseye-backports btop

## Searching

You can search the backports repositories by specifying when using `apt search` commands:

    apt search -t bullseye-backports foobar


See also: [https://backports.debian.org/Instructions/](https://backports.debian.org/Instructions/)


## Backports Kernel

There are times where you may need to install a newer kernel than Debian provides you with. For example, some hardware devices need newer kernels for full functionality. Other times, your machine may have a bug in the drivers provided by this kernel version. All are valid reasons to install a backported kernel.

You can install the 'testing' kernel to your system by installing from backports:

    sudo apt install -t bullseye-backports linux-image-amd64

It should be noted that this pulls in an additional package, so it does _not_ replace the stable kernel. Instead, a new kernel is made available. You can still boot the stable kernel from 'Advanced Options' in grub. 

```
$ apt list --installed | grep '^linux-image'
linux-image-5.10.0-13-amd64/stable,now 5.10.106-1 amd64 [installed,automatic]
linux-image-5.16.0-0.bpo.4-amd64/bullseye-backports,now 5.16.12-1~bpo11+1 amd64 [installed,automatic]
linux-image-amd64/bullseye-backports,now 5.16.12-1~bpo11+1 amd64 [installed]

$ ls -la /boot/vmlinuz*
-rw-r--r-- 1 root root 6840768 Mar 17 11:40 /boot/vmlinuz-5.10.0-13-amd64
-rw-r--r-- 1 root root 7343328 Mar  8 14:36 /boot/vmlinuz-5.16.0-0.bpo.4-amd64
```

The stable kernel will stay installed unless it is explicitly removed, which I would not recommend.
