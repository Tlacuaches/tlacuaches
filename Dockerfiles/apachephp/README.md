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

[Dockerfile](https://github.com/Tlacuaches/tlacuaches/blob/master/Dockerfiles/apachephp/Dockerfile "Ismael's Dockerfile")

### apache-config.conf:

[apache-config.php](https://github.com/Tlacuaches/tlacuaches/blob/master/Dockerfiles/apachephp/apache-config.conf "Ismael's apacheconfig")

### www/index.php:

[www/index.php](https://github.com/Tlacuaches/tlacuaches/blob/master/Dockerfiles/apachephp/www/index.php "Ismael's index.php")

### My docker-compose:

[docker-compose.yml](https://github.com/Tlacuaches/tlacuaches/blob/master/Dockerfiles/apachephp/docker-compose.yml "Ismael's docker-compose.yml")

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


> Thanks a lot Tlacuaches Project.
