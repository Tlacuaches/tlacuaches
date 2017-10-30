# Deploying VoteApp on Docker Swarm 

Well, here you have my steps:

I'm using two nodes:

**node1** called **tlacuache1** used as a **manager**.
**node2** called **tlacuachebaby**  used as a **worker**. 

## Let's get started Docker Swarm on tlacuache1:
`docker swarm init --advertise-addr IPtlacuache1`

![Image of swarm init](https://farm5.staticflickr.com/4487/37897195686_371a8b4834_b.jpg)


Remember if you don't specify a port after IPtlacuache1:port docker automatically use 2377 by default.

## Join tlacuachebaby to this swarm:

on tlacuachebay:

Use the TOKEN created when you started swarm on tlacuache1 just the same instruction.

![Image of swarm init](https://farm5.staticflickr.com/4489/26175340039_09f8b0aba0_b.jpg)

As you can see we have tlacuache1 and tlacuachebaby as part of swarm:

![Image of swarm init](https://farm5.staticflickr.com/4478/37659051851_0c9a3a81a9_b.jpg)


## The architecture of the VoteApp

![Image of swarm init](https://raw.githubusercontent.com/dockersamples/example-voting-app/master/architecture.png)
[see more details](https://github.com/dockersamples/example-voting-app "DockerExample")

**Voting-app**: Front-end Python webapp that enables a user to choose between a cat and a dog
**Redis**: Database where votes ares stored
**.NET worker**: Service that get votes from redis and store the results in a postgres database
**db**: The postgres database in wich vote's result are stored an retrieved from the result front-end
**Node.js result-app**: Front-end displaying the results of the vote. 

## Compose File
Here you have my compose file: 

[see compose file](https://github.com/tuxisma/tlacuaches/blob/48268c72fd6eaa7ebb0e1b7206490a2181c05cac/Dockerfiles/voteapp/docker-stack.yml "ComposeFile")

```
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:

```

## A tiny explication about compose file.

In this file you can see 6 services, I'm deploying another else called visualizer wich allow you displays Docker services runnig on a Docker Swarm in a diagram, I have to tell that Visualizer works only with Docker Swarm mode. 

### Networks and services

As you can see in my compose file I'm using yaml version 3, I've created two networks in order to connect my services, for example if you see the service redis I've created there a network called frontend that's why I'm using the same network in vote service as well.

The same situation with db and result both are using the same network called backend in ordert to connect those services.

### Placement

When I use placement I'm ensuring that this service only ever runs on a swarm manager - never a worker. Look those services like db, worker and visualizer.
```
placement:
        constraints: [node.role == manager]
```

### volumes
I'm using the key volumes because I need to store the information and persist.

Named value
```
volumes:
      - db-data:/var/lib/postgresql/data
```

Just specify a path and let the Docker Engine create a volume. 
```
volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
```

Let's get started the party!!!

## Deploying
`docker stack deploy -c docker-compose.yml voteapp `

![Image of swarm init](https://farm5.staticflickr.com/4467/24124215998_c95f28b65b_b.jpg)


I've deployed 6 services throught two nodes.

Let's see how many containers we have on tlacuache1:

![Image of swarm init](https://farm5.staticflickr.com/4476/26199955829_a00bc708a3_b.jpg)

and tlacuachebaby:

![Image of swarm init](https://farm5.staticflickr.com/4508/37945064472_147f92af77_b.jpg)

we have 7 containers running succesfully.

What about services:
![Image of swarm init](https://farm5.staticflickr.com/4461/37975717471_7fa6e25161_b.jpg)

And my stack:
![Image of swarm init](https://farm5.staticflickr.com/4508/37922357696_5c5f6a4198.jpg)


Now let´s check in our browser:
I'm gonna use the Tlacuache1's IP.

Visualizer is running on port 8080
![Image of swarm init](https://farm5.staticflickr.com/4504/37266123454_c4822c0b6f_b.jpg)

VoteApp is running on port 5000
![Image of swarm init](https://farm5.staticflickr.com/4492/24124694258_d93419c185.jpg)

VoteApp result is running on port 5001
![Image of swarm init](https://farm5.staticflickr.com/4443/37922441666_c8f4bd0a58.jpg)

If you use the **Tlacuachebabys'IP** you will see the glorius power of docker swarm, better said **Routing Mesh**, because It doesn´t matter the IP that you type the services always are listening.


![Image of swarm init](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)
[See more details](https://docs.docker.com/engine/swarm/ingress/#configure-an-external-load-balancer "Routing Mesh")

> Thanks Tlacuaches Project!

