Swarm is the cluster managment and orchestration features native from Docker, you can manipulate service and deploy lots of container at the same time when you use swarm mode on Docker.  

#  How to install swarm:

Install Docker 1.12.0 or later , now we are using Docker 17.09 so if you install Docker: `apt-get install docker.io` as you can see you will realize that you have Docker 17.09
 
I will show you how to deploy service in order to understand how swarm works. 

## Install Docker, see the next link 
[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)


## Setting Up the project with the next configuration:
./Dockerfile
./apache-config.conf
./www/index.php
/docker-compose.yml
### My Dockerfile:

```
FROM ubuntu:latest
MAINTAINER Ismael Garcia <tuxisma@gmail.com>

# Install apache, PHP, and supplimentary programs. openssh-server, curl, and lynx-cur are for debugging the container.
RUN apt-get update && apt-get -y upgrade && DEBIAN_FRONTEND=noninteractive apt-get -y install \
    apache2 php7.0 php7.0-mysql libapache2-mod-php7.0 curl lynx-cur

# Enable apache mods.
RUN a2enmod php7.0
RUN a2enmod rewrite

# Update the PHP.ini file, enable <? ?> tags and quieten logging.
RUN sed -i "s/short_open_tag = Off/short_open_tag = On/" /etc/php/7.0/apache2/php.ini
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php/7.0/apache2/php.ini

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

# Expose apache.
EXPOSE 80

# Copy this repo into place.
ADD www /var/www/site

# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/000-default.conf

# By default start up apache in the foreground, override with /bin/bash for interative.
CMD /usr/sbin/apache2ctl -D FOREGROUND

```


### apache-config.conf:

```
<VirtualHost *:80>
  ServerAdmin me@mydomain.com
  DocumentRoot /var/www/site

  <Directory /var/www/site/>
      Options Indexes FollowSymLinks MultiViews
      AllowOverride All
      Order deny,allow
      Allow from all
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

### www/index.php:

```
<html>
  <head><title>Tlacuache1 running</title></head>
  <body>
        <center><h1>Hello Tlacuaches!</h1></center>
        <img src="https://pre00.deviantart.net/c124/th/pre/f/2016/177/6/1/un_tlacuache_casual_g__by_supercrazyhyena-da7sdpj.png" />
        <?php
                echo "This comment is runnig the server side";
        ?>
  </body>
</html>

```

### My docker-compose:

```

version: "3"
services:
   web:
       image: tuxisma/apachephp:latest
       deploy:
          replicas: 20
          resources:
            limits:
              cpus: "0.1"
              memory: 50M
          restart_policy:
            condition: on-failure
       ports:
            - "8080:80"
       networks:
            - webnet
networks:
    webnet:

 ```

### Start Docker swarm: 
`    docker swarm init --advertise-addr IP
`
![alt text](https://farm5.staticflickr.com/4502/37643740841_138bce687e_b.jpg "docker swarm init")

As you can see we started docker swarm as a leader to deploy a service.

Let's use another node called tlacuachebaby in order to show how nodes work, tlacuache1 is my first node running like a leader and tlacuachebaby my second node is a worker.As you can notice I didn't use a specific port so Docker Swarm will use port 2377 as a port by default. 

### Add a worker tho this swarm 

Logged in tlacuachebaby:
docker swarm join --token SWMTKN-1-3oluxaye8s8denuv8ugo602lb99as6qos5rifwtp39kkn69fwg-7qsmgtk1vjriuqghvclj9mpy5 IP:2377

Note: IP --> You have to use a IP  to communicate both nodes. 

Into your swarm manager:

`
docker node ls
`
![alt text](https://farm5.staticflickr.com/4478/37659051851_9cf5646a67_b.jpg "docker node ls")


### Deploy service:
`docker stack deploy -c docker-compose.yml app
`
app --> name of service.

![alt text](https://farm5.staticflickr.com/4460/37659259391_784441449b_b.jpg "docker swarm")

It's amazing, we've  just deployed 20 containers and balance them throught two nodes, you can edit docker-compose.yml in order to redeploy service in a nutshell scale your service. 

### checkout your IP using your browser:

![alt text](https://farm5.staticflickr.com/4480/37609660866_074bd9b89f.jpg "docker swarm")
 
**Try to stop any container and you will see the magic!**
**Magically Docker Swarm create again a new one container thanks to restart_policy: condition: on-failure**


> This post was inspired by Tlacuaches Project
