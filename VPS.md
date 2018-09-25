
# VPS

## Config

passwd

## Basics

    apt-get install htop nginx -y
    snap install lxd

> Esto puede que no sea necesario! App y API corren en dockers con node
> preinstalado XD

	
## Node

    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
    apt-get install -y nodejs

## Global Node Packages

    npm i -g cross-env nvm pm2 bower sass babel-cli

#	LXC CONTAINERS CONFIG
Información sobre la creación de contenedores LXC / LXD
##	Docker
### Install Docker

    apt update
    apt install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
    sudo apt update
    apt-cache policy docker-ce
    sudo apt install docker-ce
    sudo systemctl status docker

### Install docker compose

    sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

### Docker Security Nesting (Crear y arrancar) 

    lxc launch ubuntu my-container -c security.nesting=true -c security.privileged=true

### Arrancar con docker compose (.yml)
docker-compose up
[Docker compose](https://docs.docker.com/compose/install/#master-builds)

## Crear imagen a partir de contenedor

    lxc publish neat-tadpole --alias=alpine-docker

## Enlaces

###  [Docker inside LXC](https://blog.ubuntu.com/2015/10/30/nested-containers-in-lxd)

### [Operaciones con contenedores](https://linuxcontainers.org/lxd/getting-started-cli/)


