# LX(C|D)
## Install

    apt install lxd lxd-client
    sudo lxd init

## Show running containers

    lxc list
## Show local images list

In LXC, this image server can be used by selecting the "lxc-download" template. In LXD, this image server is reachable through the "images:" default remote:

    lxc image list images:
## Launch a container
```
lxc launch ubuntu:16.04 first 
```
[More info](https://linuxcontainers.org/lxd/getting-started-cli/)
