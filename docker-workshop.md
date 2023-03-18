# Docker
## Big Picture
### The bad old days
Every time the business needed a new application, the IT department would buy a new server
### Hello VMware!
OS tax, or VM tax 

Every OS you install consumes resources!

### Hello Containers!
For a long time, the big web-scale players, like Google, have been using container technologies

containers on a single host share the host’s OS

OS Virtualization

### Linux containers
 kernel namespaces
 control groups
 union filesystems 

 containers remained complex and outside of the reach of most organizations.

 It wasn’t until Docker came

### Hello Docker!
Docker was the magic that made Linux containers usable for all

### Windows containers
windows 10 and later, and Windows Server 2016 and later

Docker Desktop running on Windows has two modes — “Windows containers” and “Linux containers”

### Mac containers
Docker Desktop
running your containers inside of a lightweight Linux VM on your Mac

### What about Kubernetes
de facto orchestrator of containerized apps

## Docker
1. Docker, Inc. the company
2. Docker the technology

### Docker, Inc.
San Francisco based technology company founded by French-born American developer and entrepreneur Solomon Hykes

PaaS provider called dotCloud. Behind the scenes, the
dotCloud platform was built on Linux containers.

they built an in-house tool that they eventually nick-named “Docker”.

the word “Docker” comes from a British expression meaning dock worker somebody who loads and unloads cargo from ships.

## The Docker technology
1. The runtime
2. The daemon (a.k.a. engine)
3. The orchestrator
   
## The Open Container Initiative (OCI)
The OCI is a governance council responsible for standardizing the low-level fundamental components of container infrastructure

1. image format : https://github.com/opencontainers/image-spec
2. container runtime : https://github.com/opencontainers/runtime-spec

these two standards is rail tracks

## Installing Docker
- Docker Desktop installs on
  - Windows 10
  - Mac
- Server installs on
  - Linux
  - Windows Server 2019
- Play with Docker : https://labs.play-with-docker.com/
  
```bash
$ docker version
```
Server:

OS/Arch: linux/amd64

Switch to Windows containers.... 

## The Ops Perspective
```bash
$ docker image ls
```
```bash
$ docker image pull ubuntu:latest
```

```bash
$ docker container run -it ubuntu:latest /bin/bash
```

```pwsh
$ docker container run -it mcr.microsoft.com/powershell:lts-nanoserver-1903 pwsh.exe
```

```bash
root@6dc20d508db0:/# ps -elf
```
```pwsh
PS C:\> ps
```
```bash
$ docker container ls
```
```bash
$ docker container exec -it vigilant_borg bash
```
```pwsh
> docker container exec -it pensive_hamilton pwsh.exe
```

```bash
$ docker container ls -a
```
## The Dev Perspective

```bash
$ git clone https://github.com/nigelpoulton/psweb.git
```

Dockerfile
```
FROM alpine
LABEL maintainer="nigelpoulton@hotmail.com"
RUN apk add --update nodejs nodejs-npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```

```bash
$ docker image build -t test:latest .
```
```bash
$ docker image ls
```

```bash
$ docker container run -d \
--name web1 \
--publish 8080:8080 \
test:latest
```

## The Docker Engine
coomponents that make up the Docker engine are; 
1. the Docker daemon
2. containerd
3. runc
4. various plugins such as networking and storage

Old days

1. The Docker daemon (hereaer referred to as just “the daemon”)
2. LXC

Docker 0.9 refactor to

2. libcontainer: pltform-agnostic tool 

### Getting rid of the monolithic Docker daemon
1. It’s hard to innovate on
2. It got slower
3. It wasn’t what the ecosystem wanted

### The influence of the Open Container Initiative (OCI)
As of Docker 1.11 (early 2016)
1. runc: reference implementation of the OCI container-runtime-spec
2. containerd: all of the container execution logic was
ripped out and refactored into
    manage container lifecycle operations — start | stop | pause | rm

### Starting a new container  
```bash
$ docker container run --name ctr1 -it alpine:latest sh
```
/var/run/docker.sock

\pipe\docker_engine

And REST API

### What’s this shim all about?
- Keeping any STDIN and STDOUT streams open so that when the daemon is restarted, the container
doesn’t terminate due to pipes being closed etc.
- Reports the container’s exit status back to the daemon.

### How it’s implemented on Linux
- dockerd (the Docker daemon)
- docker-containerd (containerd)
- docker-containerd-shim (shim)
- docker-runc (runc)

### Securing client and daemon communication
1. daemon mode: the daemon only to allow connections from clients with a valid certificate

2. client mode: client only to connect with daemons that have a valid certificate

daemon.json

/etc/docker/

C:\ProgramData\Docker\config\

```json
{
"hosts": ["tcp://node3:2376"],
"tls": true,
"tlsverify": true,
"tlscacert": "/home/ubuntu/.docker/ca.pem",
"tlscert": "/home/ubuntu/.docker/cert.pem",
"tlskey": "/home/ubuntu/.docker/key.pem"
}
```

Linux systems running systemd don’t allow you to use the “hosts” option in daemon.json. Instead,
you have to specify it in a systemd override file.
```bash
$ systemctl edit docker
```

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://node3:2376
```

```bash
$ ps -elf | grep dockerd
```

```bash
$ docker -H tcp://node3:2376 version
```

```bash
$ export DOCKER_HOST=tcp://node3:2376
```

```bash
export DOCKER_TLS_VERIFY=1
```

## Images
packaging that contains everything required for an application to run

similar to VM templates

similar to classes

made up of multiple layers

__images__ are considered __build-time__ constructs, whereas __containers__ are __run-time__ constructs

### Images are usually small
but not windows container

Dockerfile practices 
### Pulling images

/var/lib/docker/<storage-driver>

C:\ProgramData\docker\windowsfilter

Docker Desktop, everything runs inside of a VM.

```bash
$ docker image ls
```

```bash
$ docker image pull redis:latest
```

```pwsh
> docker image pull mcr.microsoft.com/powershell:latest
```
### Image naming

#### Image registries
Docker Hub (https://hub.docker.com)

https://index.docker.io/v1/

```bash
$ docker info
```
#### Official and unofficial repositories

- nginx: https://hub.docker.com/_/nginx/
- busybox: https://hub.docker.com/_/busybox/
- redis: https://hub.docker.com/_/redis/
- mongo: https://hub.docker.com/_/mongo/

unofficial
- nigelpoulton/tu-demo — https://hub.docker.com/r/nigelpoulton/tu-demo/
- nigelpoulton/pluralsight-docker-ci — https://hub.docker.com/r/nigelpoulton/pluralsight-docker-ci/

```bash
$ docker image pull <repository>:<tag>
```

```bash
$ docker image pull mongo:4.2.6
```

```bash
$ docker image pull nigelpoulton/tu-demo:v2
```

```bash
$ docker image pull gcr.io/google-containers/git-sync:v3.1.5
```

#### Images with multiple tags
```bash
$ docker image pull -a nigelpoulton/tu-demo
```

#### Filtering the output of docker image ls

```bash
$ docker image ls --filter dangling=true
REPOSITORY TAG IMAGE ID CREATED SIZE
<none> <none> 4fd34165afe0 7 days ago 14.5MB
```

- dangling
- before
- since
- label

### Searching Docker Hub from the CLI
```bash
$ docker search nigelpoulton
```
```bash
$ docker search alpine --filter "is-official=true"
```

### Images and layers

```bash
$ docker image inspect ubuntu:latest
```

#### Sharing image layers

#### Pulling images by digest

```bash
$ docker image ls --digests alpine
```

```bash
$ docker image pull alpine@sha256:9a839e63da...9ea4fb9a54
```

### Multi-architecture images
Arch ARM, x64, PowerPC, and s390x
Platform Linux Windows

- manifest lists
- manifests

```bash
$ docker container run --rm golang go version
```

```bash
$ docker manifest inspect golang | grep 'architecture\|os'
```

### Deleting Images
```bash
$ docker image rm 02674b9cb179
```
```bash
$ docker image rm $(docker image ls -q) -f
```

## Containers
```bash
$ docker container run -it ubuntu /bin/bash
```

```pwsh
> docker container run -it mcr.microsoft.com/powershell:nanoserver pwsh.exe
```

```bash
$ docker container run -it alpine:latest sleep 100
$ docker container stop
$ docker container start
```

-it flags

### Containers vs VMs
### The VM tax

### Container processes
```bash
root@50949b614477:/# ps -elf
```

killing the main process in the container will kill the container

```bash
$ docker container ls
```

```bash
$ docker container stop 50949b614477
```

```bash
$ docker container rm 50949b614477
```
### Container lifecycle

```bash
$ docker container run --name percy -it ubuntu:latest /bin/bash
```

```bash
root@9cb2d2fd1d65:/# cd tmp
root@9cb2d2fd1d65:/tmp# echo "Sunderland is the greatest football team in the world" > newfile
```

```bash
$ docker container stop percy
$ docker container ls
$ docker container ls -a
$ docker container start percy
$ docker container exec -it percy bash
```

Containers are designed to be immutable objects and it’s not a good practice to write data to them

### Stopping containers gracefully
```bash
docker container stop
docker container rm <container> -f
```

### Self-healing containers with restart policies
- always
- unless-stopped
- on-failed

```bash
$ docker container run --name neversaydie -it --restart always alpine sh
```

always and unless-stopped

unless-stopped policy will not be restarted when the daemon restarts if they were in the Stopped (Exited) state

## Containerizing an app
1. Start with your application code and dependencies
2. Create a Dockerfile that describes your app, its dependencies, and how to run it
3. Feed the Dockerfile into the docker image build command
4. Push the new image to a registry (optional)
5. Run container from the image


### Containerize a single-container app
```bash
$ git clone https://github.com/nigelpoulton/psweb.git
$ cd psweb
$ cat Dockerfile
```
```
FROM alpine
LABEL maintainer="nigelpoulton@hotmail.com"
RUN apk add --update nodejs nodejs-npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```

```bash
$ docker image build -t web:latest .
$ docker image ls
$ docker image inspect web:latest
```

### Dockerfile instructions

#### CMD
spcifies the command to run when a container is launched. It
is similar to the RUN instruction but at runtime
```
CMD ["/bin/bash", "-l"]
```
#### ENTRYPOINT
when we want a container to behave in a certain way. 
```
ENTRYPOINT ["/usr/sbin/nginx"]
CMD ["-h"]
```
```bash
$ sudo docker run -t -i jamtur01/static_web -g "daemon off;"
```

#### WORKDIR
```
WORKDIR /opt/webapp/db
RUN bundle install
WORKDIR /opt/webapp
ENTRYPOINT [ "rackup" ]
```
```bash
$ sudo docker run -ti -w /var/log ubuntu pwd
```

#### ENV
```
ENV RVM_PATH=/home/rvm RVM_ARCHFLAGS="-arch i386"
```
```bash
$ sudo docker run -ti -e "WEB_PORT=8080" ubuntu env
```

#### ADD
```
ADD software.lic /opt/application/software.lic
ADD http://wordpress.org/latest.zip /root/wordpress.zip
ADD latest.tar.gz /var/www/wordpress/
```

#### COPY
```
COPY conf.d/ /etc/apache2/
```

#### LABEL
```
LABEL version="1.0"
LABEL location="New York" type="Data Center" role="Web Server"
```
```bash
$ sudo docker inspect jamtur01/apache2
```

### Pushing images
```bash
$ docker login
```

Docker needs following information when pushing an image
- Registry
- Repository
- Tag

```bash
$ docker image tag web:latest nigelpoulton/web:latest
$ docker image ls
$ docker image push nigelpoulton/web:latest
```

```
$ docker image history web:latest
$ docker image inspect web:latest
```
```json
<Snip>
},
"RootFS": {
"Type": "layers",
"Layers": [
"sha256:cd7100...1882bd56d263e02b6215",
"sha256:b3f88e...cae0e290980576e24885",
"sha256:3cfa21...cc819ef5e3246ec4fe16",
"sha256:4408b4...d52c731ba0b205392567"
]
},
```

### Multi-stage Builds
```
FROM node:latest AS storefront
WORKDIR /usr/src/atsea/app/react-app
COPY react-app .
RUN npm install
RUN npm run build
FROM maven:latest AS appserver
WORKDIR /usr/src/atsea
COPY pom.xml .
RUN mvn -B -f pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve
COPY . .
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests
FROM java:8-jdk-alpine AS production
RUN adduser -Dh /home/gordon gordon
WORKDIR /static
COPY --from=storefront /usr/src/atsea/app/react-app/build/ .
WORKDIR /app
COPY --from=appserver /usr/src/atsea/target/AtSea-0.0.1-SNAPSHOT.jar .
ENTRYPOINT ["java", "-jar", "/app/AtSea-0.0.1-SNAPSHOT.jar"]
CMD ["--spring.profiles.active=postgres"]
```

```bash
$ git clone https://github.com/nigelpoulton/atsea-sample-shop-app.git
$ cd atsea-sample-shop-app/app
$ docker image build -t multi:stage .
$ docker image ls
```

### A few best practices

Leverage the build cache
```
FROM alpine
RUN apk add --update nodejs nodejs-npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```
Firstly, as soon as any instruction results in a cache-miss (no layer was found for that instruction), the cache is
no longer used for the rest of the entire build.

Squash the image

Add the --squash flag to the docker image build

Use no-install-recommends 

use the no-install-recommends flag with the apt-get install command

## Deploying Apps with Docker Compose
Modern cloud-native apps are made of multiple smaller services that interact

### Compose background
In the beginning was Fig

created by a company called Orchard

It was a Python tool

https://github.com/compose-spec/compose-spec


### Installing Compose on Linux
```
$ sudo curl -L \
"https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose
```

```bash
$ sudo chmod +x /usr/local/bin/docker-compose
```

```bash
$ docker-compose --version
```

```yaml
version: "3.8"
services:
  web-fe:
    build: .
    command: python app.py
    ports:
      - target: 5000
        published: 5000
    networks:
      - counter-net
    volumes:
      - type: volume
        source: counter-vol
        target: /code
  redis:
    image: "redis:alpine"
    networks:
      counter-net:
networks:
  counter-net:
volumes:
  counter-vol:
```

### Deploying an app with Compose
```bash
$ git clone https://github.com/nigelpoulton/counter-app.git
$ cd counter-app
$ ls
```
```bash
$ docker-compose up
```

```bash
$ docker-compose -f prod-equus-bass.yml up
```

```bash
docker-compose up -d
```
```bash
cat Dockerfile
```
```
FROM python:alpine << Base image
ADD . /code << Copy app into image
WORKDIR /code << Set working directory
RUN pip install -r requirements.txt << Install requirements
CMD ["python", "app.py"] << Set the default app
```
```bash
$ docker-compose down
```

```bash
$ docker-compose ps
$ docker-compose top
$ docker-compose stop
$ docker-compose restart
$ docker-compose rm
```

```bash
$ docker volume inspect counter-app_counter-vol | grep Mount
```

## Docker Swarm
Swarm groups one or more Docker nodes and lets you manage them as a cluster

Docker Swarm competes directly with Kubernetes

consists of one or more Docker nodes

Nodes are configured as managers or workers


you need the following ports open

- 2377/tcp: for secure client-to-swarm communication
- 7946/tcp and udp: for control plane gossip
- 4789/udp: for VXLAN-based overlay networks


### Initializing a new swarm
```bash
$ docker swarm init \
--advertise-addr 10.0.0.1:2377 \
--listen-addr 10.0.0.1:2377
```
```bash
$ docker node ls
```

```bash
$ docker swarm join-token worker
$ docker swarm join-token manager
```

```bash
$ docker swarm join \
--token SWMTKN-1-0uahebax...c87tu8dx2c \
10.0.0.1:2377 \
--advertise-addr 10.0.0.4:2377 \
--listen-addr 10.0.0.4:2377
```

```bash
$ docker node ls
```

### Swarm manager high availability (HA)
1. Deploy an odd number of managers.
2. Don’t deploy too many managers (3 or 5 is recommended)

#### split-brain

### Swarm services
services are a new construct introduced with Docker 1.12, and they only apply to swarm mode

1. Imperatively on the command line with docker service create
2. Declaratively with a stack file

```bash
$ docker service create --name web-fe \
-p 8080:8080 \
--replicas 5 \
nigelpoulton/pluralsight-docker-ci
```

```bash
$ docker service ls
$ docker service ps web-fe
```

```bash
$ docker service inspect --pretty web-fe
```

### Replicated vs global services
--mode global

### Scaling a service
```bash
$ docker service scale web-fe=10
```
```bash
$ docker service scale web-fe=5
```
#### Removing a service
```bash
$ docker service rm web-fe
```

#### Rolling updates
```bash
$ docker network create -d overlay uber-net
$ docker network ls
```
```bash
$ docker service create --name uber-svc \
--network uber-net \
-p 80:80 --replicas 12 \
nigelpoulton/tu-demo:v1
```

ingress mode vs host mode
```bash
$ docker service create --name uber-svc \
--network uber-net \
--publish published=80,target=80,mode=host \
--replicas 12 \
nigelpoulton/tu-demo:v1
```

```bash
$ docker service update \
--image nigelpoulton/tu-demo:v2 \
--update-parallelism 2 \
--update-delay 20s uber-svc
```

```bash
$ docker service inspect --pretty uber-svc
```
### Troubleshooting
```bash
$ docker service logs svc1
```

### Backing up Swarm

/var/lib/docker/swarm

```bash
$ service docker stop
$ tar -czvf swarm.bkp /var/lib/docker/swarm/
```

All swarm nodes must have their Docker
daemon stopped and the contents of their /var/lib/docker/swarm directories deleted.

1. You can only restore to a node running the same version of Docker the backup was performed on
2. You can only restore to a node with the same IP address as the node the backup was performed on

```bash
$ tar -zxvf swarm.bkp -C /
$ service docker start
$ docker swarm init --force-new-cluster
```

## Docker Networking
no network, no app!



### The theory
- The Container Network Model (CNM)
- libnetwork
- Drivers
  - Linux  
    - bridge
    - overlay
    - macvlan
  - Windows
    - nat 
    - overlay
    - transparent

### The Container Network Model (CNM)
- Sandboxes
- Endpoints
- Networks

### Single-host bridge networks

```bash
$ docker network ls
$ docker network inspect bridge
$ docker network inspect bridge | grep bridge.name
```

```bash
$ docker network create -d bridge localnet
```

```bash
$ brctl show
```

```bash
$ docker container run -d --name c1 \
--network localnet \
alpine sleep 1d
```

```bash
$ brctl show
```

```bash
$ docker container run -d --name web \
--network localnet \
--publish 5000:80 \
nginx
```

```bash
$ docker port web
```

### Connecting to existing networks
The built-in MACVLAN driver (transparent on Windows)
```bash
$ docker network create -d macvlan \
--subnet=10.0.0.0/24 \
--ip-range=10.0.0.0/25 \
--gateway=10.0.0.1 \
-o parent=eth0.100 \
macvlan100
```

```bash
$ docker container run -d --name mactainer1 \
--network macvlan100 \
alpine sleep 1d
```
### Multi-host overlay networks

-d overlay

### Container and Service logs for troubleshooting
```bash
$ journalctl -u docker
```

(daemon.json) so that “debug” is set to “true” and “log-level”
- debug The most verbose option
- info The default value and second-most verbose option
- warn Third most verbose option
- error Fourth most verbose option
- fatal Least verbose option

```json
{
<Snip>
"debug":true,
"log-level":"debug",
<Snip>
}
```

### Service discovery
```bash
$ docker container run -it --name c1 \
--dns=8.8.8.8 \
--dns-search=nigelpoulton.com \
alpine sh
```
service discovery is network-scoped 


### Ingress load balancing
Swarm supports two publishing modes

- Ingress mode (default)
- Host mode

```bash
$ docker service create -d --name svc1 \
--publish published=5000,target=80,mode=host \
nginx
```






