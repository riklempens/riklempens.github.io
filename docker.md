# Technote Docker

## Docker basic commands

Docker file example:
```
FROM debian
MAINTAINER (https://www.danieldent.com/)
RUN DEBIAN_FRONTEND=noninteractive apt-get update -q && \
    DEBIAN_FRONTEND=noninteractive apt-get dist-upgrade -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y wget build-essential bind9utils libbind-dev libkrb5-dev libssl-dev libcap-dev libxml2-dev dnsutils && \
    wget ftp://ftp.nominum.com/pub/nominum/dnsperf/2.0.0.0/dnsperf-src-2.0.0.0-1.tar.gz && \
    echo "23d486493f04554d11fca97552e860028f18c5ed6e35348e480a7448fa8cfaad dnsperf-src-2.0.0.0-1.tar.gz" | sha256sum -c && \
    tar -xvvzf dnsperf-src-2.0.0.0-1.tar.gz && \
    cd dnsperf-src-2.0.0.0-1 && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -Rf dnsperf-src-2.0.0.0-1 dnsperf-src-2.0.0.0-1.tar.gz && \
    apt-get autoremove -y build-essential && \
    rm -Rf /var/lib/apt /var/cache/apt
```

### Docker build

```
docker build --no-cache -t <NAME>:<VERSION> <DOCKERFILE>
```

### Docker run
Run command with an interactive shell:
```
docker run -it --rm <IMAGE> bash
```

Use an alias to turn a container into a command:
```
alias traceroute='docker run -t --rm transceptortechnology/traceroute'
```

### Docker socket
you can manage your host containers from within a container by mounting the Docker socket.
```
docker run -it -v /var/run/docker.sock:/var/run/docker.sock\ ubuntu:latest sh -c "apt-get update ; apt-get install docker.io\
-y ; bash"
```

## Docker autoupdate

Watchtower helps to autoupdate containers
[GitHub - CenturyLinkLabs/watchtower: Automatically update running Docker containers](https://github.com/CenturyLinkLabs/watchtower)

Using in docker-compose:

```
  watchtower:
    image: centurylink/watchtower
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
    restart: always
```

### Docker cleanup

Delete exited containers:
```
docker rm -v $(docker ps -a -q -f status=exited)
```

*Docker keeps all of the images that you have used in the disk, even if those are not actively running. This is done so that it has the necessary images in the local ‘cache’. This is awesome because when you want to pull an image that depends on those, or when you are building an image, all of these are locally available. The bad news is that this eats up disk space!*

The command to remove these unwanted images is:
```
docker rmi $(docker images -f "dangling=true" -q)
```

Remote  ‘dangling’ volumes:
```
docker volume rm $(docker volume ls -qf dangling=true)
```

## Docker compose
To rebuild an image you must use
```
docker-compose build` or `docker-compose up --build
```

## Docker swarm
Just as you run a single container with docker run, you can now start a replicated, distributed, load balanced process on a swarm of Engines with docker service:

```
docker service create –name frontend –replicas 5 -p 80:80/tcp nginx:latest
```

This command declares a desired state on your swarm of 5 nginx containers, reachable as a single, internally load balanced service on port 80 of any node in your swarm.
See also: https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/

##  Install docker-ce on ubuntu 17.04
Add the following lint to /etc/apt/sources.list:

```
deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable
```

Run the following commands to install docker-ce:
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7EA0A9C3F273FCD8
sudo apt update
sudo apt-get install docker-ce
```
