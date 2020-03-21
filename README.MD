# Devops on Docker
This is probably your first Devops solution.

## Quick start
Let me briefly describe the behavior that basic Devops should have:  

*Coding >> Commit >> Hooks >> Building >> Unit Testing >> Image Pushing >> Deploying*

Obviously, Devops is composed of multiple systems. They carried out division of labor and cooperation, as well as pipelined operations, and finally achieved the results we wanted.

Therefore, I have selected several simple and lightweight open source projects in the open source community to form our Devops components.

### Gitea
As a private GIT repository, Gitea is portable enough, it has only one binary file, and all the functions are carried in a size less than 60MB.
##### Deploying
```bash
docker-compose -f ./Gitea/docker-compose.yaml up -d
```

### Registry
As a private docker registry, this is also the official open source private registry solution of docker. In addition, you have no choice. :(

*Need to configure according to your environment before deployment*
##### Deploying
```bash
docker-compose -f ./Registry/docker-compose.yaml up -d
```

### Drone
Now, we need a CI/CD platform to mobilize tasks. Here I choose Drone CI.

It also depends on the Go high-performance compiler. Drone is not only very convenient to deploy, but more importantly, its stable operation will only occupy less than 20MB of memory on your server.
##### Deploying
```bash
docker-compose -f ./Drone/docker-compose.yaml up -d
```

### Swarmpit
Finally, I also need a container orchestration engine to manage my services, achieve smooth service upgrades, horizontal expansion, rollback release, etc.

Regarding the container orchestration engine, swarm is actually not a mainstream choice, but it is only used in a small range and the business volume is not large. I don’t need so many complicated functions and features of K8S, so Docker own Swarm became my best choice because it was simple enough and very easy to use.
##### Deploying
```bash
docker stack deploy -c ./Swarmpit/docker-compose.yml swarmpit
```