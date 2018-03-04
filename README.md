# SLARP
**S**imple {**L**et's Encrypt + **A**pache} **R**everse **P**roxy


## Goal

SLARP is an environment that is ready to forward HTTP requests to backend web servers (so it acts as a reverse proxy), while ensuring SSL termination for HTTPS requests.  
Easily installable and sufficient in many cases.  
There's almost no configuration to be done for SLARP itself. Your main work is to write **vhosts** that call your backends, and to run some short commands to request new Let's Encrypt SSL certificates (renewal is then automatic).  
It is not especially designed for high scalability, though Apache (here used with its default configuration) can handle a decent number of simultaneous connections.

The typical situation is when:
* You have several containers on your machine that act as backend web servers, so you need a single entrypoint that listens to the ports 80/443 of the host.
* It's better if this entrypoint runs isolated in its container.
* Some of the backends must be originally accessed via HTTPS, and certified by Let's Encrypt.


## How to install and use

Here I describe commands that work on a classic debian-based Linux distribution, with a recent version of Bash.  
All operations must be run as **root**. The reasons are:
* Server administration tasks usually need the highest privileges.
* Thus, files of this project are intended to belong to root.

### 1- Get the minimal Docker image on which the SLARP image will be based

SLARP needs some things that are provided by my project [debian9-workbase](https://gitlab.zareldyn.net/zareldyn/debian9-workbase#debian9-workbase).  
Basically, all you have to do is (but you may want to check that project deeper to know what it is intended for):
```
# docker build --no-cache -t my-debian9 https://gitlab.zareldyn.net/zareldyn/debian9-workbase.git
 ```
This will build a Docker image with tag "my-debian9". You can choose another name if you want.

### 2- Clone this project

Note that the working copy is the location where some data (certificates, logs…) will persist; you may have noticed the empty directories in the project's tree.  
```
# git clone https://gitlab.zareldyn.net/zareldyn/slarp.git && ./slarp/fix-permissions
```

After this, it's time to decide whether the name "slarp" is meaningful or not for you, because the image you'll build at the next step and the resulting container will both be named after the directory containing this project's tree.  
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
Now a container named "slarp-reverse-proxy" is running, but for now it has nothing to forward.

### 5- Define your needs

Let's say your machine must handle requests like `http(s)://www.my-great-website.org/a-page`. The domain `www.my-great-website.org` is bound to your IP but you have no certificate for it, and you'd like to forward the requests to a container named `my-great-website`. Additionally, you want to redirect all HTTP requests to their HTTPS equivalent.

#### Add a vhost

In the *vhosts* directory of the SLARP working copy, add a file containing:
```apache
# This is just an example of Apache virtual host configuration.
# Following the Apache convention, this file should be named xxx-my-great-website.conf.

<VirtualHost *:80>
    ServerName www.my-great-website.org

    RewriteEngine on
    RewriteRule ^(.*)$ https://www.my-great-website.org$1 [QSA,R=301,L]

    ErrorLog ${APACHE_LOG_DIR}/my-great-website.error.log
    CustomLog ${APACHE_LOG_DIR}/my-great-website.access.log combined
</VirtualHost>

# The line below is a directive for SLARP so http://my-great-website/ (port 80 of a local container) can be reached.
#:resolve-container:my-great-website

<VirtualHost *:443>
    ServerName www.my-great-website.org

    SSLEngine on
    SSLCertificateKeyFile /etc/letsencrypt/live/www.my-great-website.org/privkey.pem
    SSLCertificateFile /etc/letsencrypt/live/www.my-great-website.org/fullchain.pem

    # Some backends need something like this, so they know that the original request was a HTTPS request.
    RequestHeader set X-Forwarded-Proto "https"

    ProxyPass / http://my-great-website/
    ProxyPassReverse / http://my-great-website/
    ProxyPreserveHost On

    ErrorLog ${APACHE_LOG_DIR}/my-great-website.error.log
    CustomLog ${APACHE_LOG_DIR}/my-great-website.access.log combined
</VirtualHost>
```
Notes:
* The internal `${APACHE_LOG_DIR}` directory is bound to the *apache-logs* directory of the SLARP working copy.
* The internal `/etc/letsencrypt` directory is bound to the *certs* directory of the SLARP working copy; always use the pattern "/etc/letsencrypt/live/{domain}/{file}" for `SSLCertificate*` directives.

#### Get a new Let's Encrypt SSL certificate

This example of virtual host mentions certificate files that don't exist yet. To generate them, enter the container you launched at step 4:
```
# docker exec -ti slarp-reverse-proxy su
```
Then, run
```
certbot certonly --apache -d www.my-great-website.org
```
and follow the instructions.  
No other command required, you can [ctrl][d] the SLARP environment.

### 6- Apply your changes

Once you have made changes in the *vhosts* directory, run
```
# ./reload
```
This command resolves the needed backend containers to IPs (it follows the directives for SLARP in the vhost files) and reloads the Apache service.

Now it's time to verify if everything is OK with `https://www.my-great-website.org` ;-)

### 7- And after

#### Reload / Restart

If a backend web server is restarted, perhaps its IP has changed. In this case, run
```
# ./reload
```
or
```
# ./resolve-containers
```
The *resolve-containers* command do only containers resolving to IPs, without reloading the Apache service.  
The *reload* command must be run when you change something in your vhosts.  
A full restart is done with:
```
# ./stop && ./start
```
The *stop* command removes the SLARP container.

#### Backup

To write…
