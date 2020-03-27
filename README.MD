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
##### Example
In addition, in order for Drone to coordinate with our own Gitea and tell it what to do after the code is submitted, we also need a YAML to describe its "work items".

github：https://github.com/otokaze/yourip
```yaml
---
kind: pipeline
type: docker
name: default

platform:
  os: linux
  arch: amd64

steps:
- name: linter
  image: alpine/git
  commands:
  - change=$(git diff origin/master $DRONE_COMMIT ./CHANGELOG.md 2> /dev/null) && code=0 || code=$?
  - if [ ! $code -eq 0 ]; then echo 'CHANGELOG.md not fount.'; exit $code; fi
  - if [ -z "$change" ]; then echo 'CHANGELOG.md no change.'; exit 1; fi

- name: builder
  image: golang:1.13.8
  environment:
    GOOS: linux
    GOARCH: amd64
    CGO_ENABLED: 0
  commands:
  - go build -o yourip
  - go test

- name: tagger
  image: alpine
  commands:
  - tags=$(grep -E -o  v[0-9]+\.[0-9]+\.[0-9]+ CHANGELOG.md | head -1 | sed s/v/latest,/g)
  - if [ -z $tags ]; then echo 'No version found in CHANGELOG.md'; exit 1; else echo $tags > .tags; fi
  when:
    event:
    - push

- name: pushing
  image: plugins/docker
  settings:
    username:
      from_secret: DOCKER_REGISTRY_USERNAME
    password:
      from_secret: DOCKER_REGISTRY_PASSWORD
    repo: registry.otokaze.cn/yourip
    registry: registry.otokaze.cn
  when:
    event:
    - push

- name: deploying
  image: curlimages/curl
  environment:
    DEPLOY_API: https://swarm.otokaze.cn/api/services/yourip/redeploy
    TOKEN:
      from_secret: DOCKER_SWARMPIT_TOKEN
  commands:
  - code=$(curl -XPOST -s -w %{http_code} "$DEPLOY_API?tag=latest" -H 'Content-Type:application/json' -H "authorization:$TOKEN")
  - if [[ $code == "" || $code -lt 200 ]]; then echo "redeploy failed. HTTP_CODE=${code}"; exit 1; fi
  when:
    event:
    - push

trigger:
  branch:
  - master
  event:
  - pull_request
  - push
...
```

### Swarmpit
Finally, I also need a container orchestration engine to manage my services, achieve smooth service upgrades, horizontal expansion, rollback release, etc.

Regarding the container orchestration engine, swarm is actually not a mainstream choice, but it is only used in a small range and the business volume is not large. I don’t need so many complicated functions and features of K8S, so Docker own Swarm became my best choice because it was simple enough and very easy to use.
##### Deploying
```bash
docker stack deploy -c ./Swarmpit/docker-compose.yml swarmpit
```