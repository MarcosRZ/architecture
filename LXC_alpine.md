
# LXC Alpine Containers 
Version 2.0 containers are based in Alpine Linux lightweight image. This wasn't possible before due to the lack of compatibility with Docker. We're no longer using docker inside LXC so, we can use Alpine as base image which is wonderful.
## Create an Alpine Edge container
lxc launch images:alpine/edge

## Install bash
Alpine comes with a minimal set of features and it is intended to be configured to fit your needs. The first thing we need is a shell.

    lxc exec container-name -- apk add bash

## Go inside the container
As we don't have SSH (yet), we need to enter the container in a LXC manner.

    lxc exec neat-tadpole -- /bin/bash

## Set root password

    passwd

## Install sshd

    apk add openssh

## Configure sshd to allow root access
Login as root, and login with password are disabled by default in openssh. 

    vi /etc/ssh/sshd_config

> set PermitRootLogin to yes
> set PasswordAuthentication to yes

## Add sshd to rc scripts (boot)

    rc-update add sshd

## Restart sshd

    service sshd restart

## More info
 [Setting up a ssh server in Linux Alpine](https://wiki.alpinelinux.org/wiki/Setting_up_a_ssh-server)

# Docker 
**Important!**
I was unable to make docker work with Alpine Linux. There was a lot of issues with cgroups and it just didn't run. Anyway, these are the theoretical instructions to make it work.

## Install Docker

    apk add docker
    rc-update add docker boot
    service docker start

## Install Docker-compose

    apk add py-pip
    pip install docker-compose

