# SLARP
**S**imple {**L**et's Encrypt + **A**pache} **R**everse **P**roxy


## Goal

An easy way to forward HTTP requests to backend web servers, while ensuring SSL termination for incoming HTTPS requests.  
Not very elaborate, but quickly installable and sufficient in many cases.  
There's almost no configuration to be done for this project itself, but your main work is to write *vhosts* that call your backends, in addition to run some short commands to request SSL certificates for the first time (the renewal is automatic).  
Thanks to Let's Encrypt / Certbot, SSL certificates are managed in a simple way.  
It is not especially designed for high scalability, though Apache (here used with its default configuration) can handle a decent number of simultaneous connections.

The typical situation is:
* You have several containers on your machine that act as web servers.
* You need to route every incoming request to the right container.
* Some/all of them must be accessed via trusted HTTPS (does not work on private networks!).
* It's better to have a single place where SSL certificates are managed.


## How to install and use

Here I describe commands that work on a classic debian-based Linux distribution.  
All operations must be run as *root*. The reasons are:
* Server administration tasks usually need the highest privileges.
* Thus, files of this project are intended to belong to root.

### 1- Get the minimal Docker image on which the reverse proxy image will be based (at step 3)

SLARP needs some things that are provided by my project [debian9-workbase](https://gitlab.zareldyn.net/zareldyn/debian9-workbase#debian9-workbase).  
Basically, all you have to do is  
`docker build --no-cache -t my-debian9 --build-arg SYSTEM_TIMEZONE=$(cat /etc/timezone) --build-arg PARENT_HOSTNAME=$(hostname) https://gitlab.zareldyn.net/zareldyn/debian9-workbase.git`  
This will build a Docker image with tag "my-debian9". You can choose another name if you want.  
This will also pull the offical debian 9.0 image (since my-debian9 is based on it), if you have not pulled it already.

### 2- Clone this project

Note that the working copy is the location where the persistent data will be saved (certificates, logs…) - you may have noticed the empty directories in the project's tree.  
`git clone https://gitlab.zareldyn.net/zareldyn/slarp.git`

After this, it's time to decide whether the name "slarp" is convenient or not for the way you manage your Docker images and containers, because the image you'll build at the next step and the resulting container will both be named after the directory containing this project's tree.  
For example, if you prefer "slarp-reverse-proxy", do this:  
`mv slarp slarp-reverse-proxy`  
so the final image will be tagged "slarp-reverse-proxy" and the container will have the same name.
