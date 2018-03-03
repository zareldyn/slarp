# SLARP
**S**imple {**L**et's Encrypt + **A**pache} **R**everse **P**roxy


## Goal

An easy way to forward HTTP requests to backend web servers, while ensuring SSL termination for HTTPS requests.  
Quickly installable and sufficient in many cases.  
There's almost no configuration to be done for this project itself. Your main work is to write *vhosts* that call your backends, and to run some short commands to request new Let's Encrypt SSL certificates (renewal is then automatic).  
It is not especially designed for high scalability, though Apache (here used with its default configuration) can handle a decent number of simultaneous connections.

The typical situation is when:
* You have several containers on your machine that act as backend web servers, so you need a single entrypoint that listens to the ports 80/443 of the host.
* It's better if this entrypoint runs isolated in its container.
* Some of the backends must be originally accessed via HTTPS.


## How to install and use

Here I describe commands that work on a classic debian-based Linux distribution.  
All operations must be run as *root*. The reasons are:
* Server administration tasks usually need the highest privileges.
* Thus, files of this project are intended to belong to root.

### 1- Get the minimal Docker image on which the SLARP image will be based

SLARP needs some things that are provided by my project [debian9-workbase](https://gitlab.zareldyn.net/zareldyn/debian9-workbase#debian9-workbase).  
Basically, all you have to do is (but you may want to check that project deeper to know what it is intended for):
```
# docker build --no-cache -t my-debian9 https://gitlab.zareldyn.net/zareldyn/debian9-workbase.git
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

### 4- Launch SLARP

Be sure you have no other program listening to the ports 80 and 443. Then, just run
```
# ./start
```

### 5- Define your needs

Let's say your machine must handle requests like *http(s)://www.my-great-website.org/a-page*. The domain *www.my-great-website.org* is bound to your IP but you have no certificate for it, and you'd like to forward the requests to a container named *my-great-website*. Additionally, you want to redirect all HTTP requests to their HTTPS equivalent.

#### Add a vhost

In the *vhosts* directory of the SLARP working copy, add a file containing:
```apache
#:resolve-container:my-great-website

<VirtualHost *:80>
    ServerName www.my-great-website.org

    RewriteEngine on
    RewriteRule ^(.*)$ https://www.my-great-website.org$1 [QSA,R=301,L]

    ErrorLog ${APACHE_LOG_DIR}/my-great-website.error.log
    CustomLog ${APACHE_LOG_DIR}/my-great-website.access.log combined
</VirtualHost>

<VirtualHost *:443>
    ServerName www.my-great-website.org

    SSLEngine on
    SSLCertificateKeyFile /etc/letsencrypt/live/www.my-great-website.org/privkey.pem
    SSLCertificateFile /etc/letsencrypt/live/www.my-great-website.org/fullchain.pem

    RequestHeader set X-Forwarded-Proto "https"
    ProxyPass / http://my-great-website/
    ProxyPassReverse / http://my-great-website/
    ProxyPreserveHost On

    ErrorLog ${APACHE_LOG_DIR}/my-great-website.error.log
    CustomLog ${APACHE_LOG_DIR}/my-great-website.access.log combined
</VirtualHost>
```

This is just an example of Apache virtual host configuration. Following the Apache convention, this file should be named *xxx-my-great-website.conf*.  
Note that:
* The first line `#:resolve-container:my-great-website` is a comment for Apache, but is a directive for SLARP. Without it, `http://my-great-website/` (port 80 of a local container) could not be reached.
* Some backends need something like `RequestHeader set X-Forwarded-Proto "https"` so they know that the original request was a HTTPS request.
* The internal `${APACHE_LOG_DIR}` is bound to the *apache-logs* directory of the SLARP working copy.
* The internal `/etc/letsencrypt` is bound to the *le-certs* directory of the SLARP working copy ; always use the pattern */etc/letsencrypt/live/{domain}/{file}.pem* for `SSLCertificate*` directives.

#### Get a new Let's Encrypt SSL certificate

To write…
