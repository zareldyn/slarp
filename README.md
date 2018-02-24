# SLARP
**S**imple {**L**et's Encrypt + **A**pache} **R**everse **P**roxy


## Goal

An easy way to forward HTTP requests to backend web servers, while ensuring SSL termination for HTTPS requests.  
Quickly installable and sufficient in many cases.  
There's almost no configuration to be done for this project itself. Your main work is to write *vhosts* that call your backends, and to run some short commands to request new Let's Encrypt SSL certificates (renewal is then automatic).  
It is not especially designed for high scalability, though Apache (here used with its default configuration) can handle a decent number of simultaneous connections.

The typical situation is when:
* You have several containers on your machine that act as web servers, so you need a single entrypoint that listens to the ports 80/443 of the host.
* It's better if this entrypoint runs isolated in its container.
* Some of the backends must be originally accessed via HTTPS.


## How to install and use

Here I describe commands that work on a classic debian-based Linux distribution.  
All operations must be run as *root*. The reasons are:
* Server administration tasks usually need the highest privileges.
* Thus, files of this project are intended to belong to root.

### 1- Get the minimal Docker image on which the SLARP image will be based

SLARP needs some things that are provided by my project [debian9-workbase](https://gitlab.zareldyn.net/zareldyn/debian9-workbase#debian9-workbase).  
Basically, all you have to do is  
```
# docker build --no-cache -t my-debian9 \
               --build-arg SYSTEM_TIMEZONE=$(cat /etc/timezone) \
               https://gitlab.zareldyn.net/zareldyn/debian9-workbase.git
 ```
This will build a Docker image with tag "my-debian9". You can choose another name if you want.  
This will also pull the official debian 9.0 image (since my-debian9 is based on it), if you have not pulled it already.

### 2- Clone this project

Note that the working copy is the location where some data (certificates, logs…) will persist - you may have noticed the empty directories in the project's tree.  
```
# git clone https://gitlab.zareldyn.net/zareldyn/slarp.git && ./slarp/fix-permissions
```

After this, it's time to decide whether the name "slarp" is adequate or not for the way you manage your Docker images and containers, because the image you'll build at the next step and the resulting container will both be named after the directory containing this project's tree.  
For example, if you prefer "slarp-reverse-proxy", do this:  
```
# mv slarp slarp-reverse-proxy
```
so the final image will be tagged "slarp-reverse-proxy" and the container will have the same name.

### 3- Build the SLARP image

With the previous examples, it would be:  
```
# cd slarp-reverse-proxy
# ./build -f my-debian9
```
This builds the "slarp-reverse-proxy" image from the "my-debian9" image.

### 4- To be continued…
