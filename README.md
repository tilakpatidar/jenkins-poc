## Jenkins POC
### Setup

```shell
# create network for jenkins
docker network create jenkins

# start docker in docker engine for submitting jenkins agent requests to it
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2

# Build the jenkins master agent
docker build -t myjenkins-blueocean:1.1 .

# Start the master jenkins agent
docker run --name jenkins-blueocean --rm --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1

```

### Start/Stop
```shell
# Stopping
docker stop jenkins-blueocean jenkins-docker

# To start again just use  docker run commands from setup step again  
```

### Interact with docker in docker (dind) engine

```shell
# This will show all jenkins jobs containers running inside dind engine
docker exec -it jenkins-docker docker ps
```