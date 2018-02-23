# SLARP
**S**imple {**L**et's Encrypt + **A**pache} **R**everse **P**roxy


## Goal

An easy way to forward HTTP requests to backend web servers, while ensuring SSL termination.  
Thanks to Let's Encrypt / certbot, SSL certificates are managed in a simple way.  
It is not especially designed for high performances, though Apache (here used with its default configuration) can serve a decent number of simultaneous connections.

The typical situation is:
* You have several containers on your machine that act as web servers.
* You need to route every incoming request to the right container.
* Some/all of them must be accessed via HTTPS.
* It's better to have a single place where SSL certificates are managed.


## How to install and use

Here I describe commands that work on a classic debian-based Linux distribution.  
All operations must be run as *root*. The reasons are:
* Server administration tasks usually need the highest privileges.
* Files of this project are intended to belong to root.

### 1- Get the minimal Docker image on which the reverse proxy image will be based

SLARP needs some things that are provided by my project [debian9-workbase](https://gitlab.zareldyn.net/zareldyn/debian9-workbase#debian9-workbase).  
Basically, all you have to do is  
`docker build --no-cache -t my-debian9 --build-arg SYSTEM_TIMEZONE=$(cat /etc/timezone) --build-arg PARENT_HOSTNAME=$(hostname) https://gitlab.zareldyn.net/zareldyn/debian9-workbase.git`  
This will add a Docker image tagged "my-debian9". Of course you can choose another name.  
This will also add the offical debian 9.0 image, if you have not pulled it already.

### 2- 

To be continued…
