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

