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
# First copy the certs for docker engine
docker cp jenkins-docker:/certs/client $HOME/dind_certs/

# Then create a docker context for dind engine
docker context create jenkins-de --docker "host=tcp://localhost:2376,ca=$HOME/dind_certs/ca.pem,cert=$HOME/dind_certs/cert.pem,key=$HOME/dind_certs/key.pem"

# Switch from your local docker engine to docker-in-docker engine
docker context use jenkins-de
docker ps

# switch back to your local docker engine
docker context use default
docker ps
```