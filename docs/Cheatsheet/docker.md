---
layout: default
title: Docker
parent: Cheatsheet
nav_order: 60
---

# Docker


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Reference

{: .no_toc }

- [Docker Tutorial for Beginners - A Full DevOps Course on How to Run Applications in Containers](https://www.youtube.com/watch?v=fqMOX6JJhGo)
- Installation Guide

## Overview

- [Concept] Docker *<u>containers</u>* share the underlying OS kernel. 

  - docker vs. VM

  |              | VM                                                   | docker                                                       |
  | ------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
  | stack layout | VM(applications, libs, deps, OS) -> Hypervisor -> HW | container(applications, libs, deps) -> docker -> OS kernel -> VM |

  >  So you won’t be able to run a windows container on a docker host with Linux kernel on it.

- [Concept] [Docker Hub](https://hub.docker.com/)

## Docker Commands

Note: 

- *a container only lives as long as the process inside it is alive.*
- *every container gets an internal IP inside <u>docker host</u> assigned by default*

```bash
docker run <image> # download the image from Docker Hub for the first time
docker run --name <container name> <image>
docker run -d <image> # detach/background mode
docker attach <NAME or ID shown by "ps"> # re-attach a running container

docker run redis:4.0 # ":4.0" is a tag specifying the version. 
docker run -i <image name>  # interactive mode(redirect host's stdin)
docker run -t <image name> # sudo terminal
rodocker run -it <image name>

# Execute commands in a docker
docker run ubuntu sleep 5
docker exec <NAME or ID shown by "ps"> <command>

# inspect containter
docker ps
docker ps -a
docker inspect <NAME or ID>

docker stop <NAME or ID shown by "ps">
docker rm <NAME or ID shown by "ps"> # removea stopped or exited container permanently

docker images # list all avaliable images on our hosts
docker pull <image> # only pull the images
docker rmi <image> # remove images; Note that you must stop and delete all dependent containers

# port mapping
docker run -p <external port>:<internal port> <image>
docker run -p 80:5000 webapp

# volume mapping(save persisted data)
docker run -v <volume name>:<internal dir> <image>
docker run -V <complete path>:<interal dir> <image> # old way of bind mount
docker run --mount type=bind,source=<complete path>,target=<internal dir> <image>

# container logs
docker log <NAME or ID>

# environment variables
docker run -e APP_COLOR=blue <name> # pass envrionment variables
docker inspect <NAME or ID> # inspect environment variables

```

## Create Customized Image (Dockerfile)

> Typical Routine to set up the environment
>
> - install OS
> - update apt repo
> - install dependencies using apt
> - install application dependencies(like pip in Python)
> - copy source code to directory(like `/opt`)
> - run the command

- [Concept] Dockerfile:

  - build each line of instructions in a <u>layered architecture</u>

    >  if we modifying one of the instructions, only the above the updated layers needs to be rebuilt.

```dockerfile
FROM <base OS or another image>

MAINTAINER magicgriffin <maijing3007@qq.com>

RUN <command run in sudo mode>

# copy files from the local system onto the docker image
COPY <dir in host> <dir in image>

USER user # affect following 'RUN', CMD', 'ENTRYPOINT' etc.

# define the default program that will be run within the container
CMD <command> <param1>
CMD ["command", "param1"] # json format

ENTRYPOINT <command> # run the command which appended parametes

# combine "ENTRYPOINT" and "CMD"
ENTRYPOINT sleep
CMD 5 # default parameters
```

```bash
docker build -t <tag name> -f <Dockerfile name> . # create locally
docker build github.com/creack/docker-firefox # build vis URL
docker push <image name> # push image to Docker Registry, like docker hub
docker run --entrypoint <command> <image> 
docker history <image name> # inspect the layered architecture
```

> `CMD` vs. `ENTRYPOINT`

Recall that *a container only lives as long as the process inside it is alive.*

- `CMD` in Dockerfile will define the default program that will be run within the container

  - `docker run <image name> <COMMAND>` will <u>override</u> the default command
- `ENTRYPOINT` is like the command instruction that you can specify the program that will be run when the container starts, and <u>appends</u> the parameters.
	- In Dockerfile: `ENTRYPOINT sleep`; and run `docker run ubuntu-sleeper 10`
  	- to override the entrypoint, leverage the `--entrypoint` flag.

## Docker Networking

- [Concept] Docker Network

  - (Default Network)<u>Bridge</u>: the *default network* a container get attached to
  - `none`: none network
  - `host`: attach to host network

  {% include img.html src="docker.assets/image-20200413222851834.png" alt="image-20200413222851834" %} 

  - User-defined networks

  {% include img.html src="docker.assets/image-20200413223138912.png" alt="image-20200413223138912" %} 

- [Concept] Embedded DNS
    - <u>container name</u> could be used as the hostname
      
      {% include img.html src="docker.assets/image-20200413223646506.png" alt="image-20200413223646506" %} 


```bash
docker run <image> --network=none
docker run <image> --network=host

docker network create --driver bridge --subnet 182.18.0.0/16 <customized network name>
docker network ls
docker inspect <container name>
```

## Docker Storage

- Storage location on Host:

  - (Linux) `var/lib/docker`

    ```bash
    /var/lib/docker
    	/aufs # storage driver on host OS
    	/containers
    	/image
    	/volumes
    ```

Recall the layered architecture. 

> image, container vs volumes
>
> - (image) each layer in a image is read-only
> - (container) when you creates a container, it create a new writable layer on top of the image. So, when the container is destroyed, this layer and all the storage are also destroyed.
> - (volume) persistent storage

```bash
docker volume create <volume name>

# volume mapping(save persisted data)
docker run -v <volume name>:<internal dir> <image>

# bind mount(arbitory mount point on host)
docker run -V <complete path>:<interal dir> <image> # old way
docker run --mount type=bind,source=<complete path>,target=<internal dir> <image>
```

## Docker Compose

purpose: set-up an entire containers stack with ease

```bash
# old way
docker run --link <internal container name>:<external container name> <image>

# new way
docker-compose up
```

{% include img.html src="docker.assets/image-20200413231134993.png" alt="image-20200413231134993" %} 

```yaml
<container name>:
	build: <a directory whih a Dockerfile in it, from which we can build the container. 
	image: <image name>
	ports:
		- <internal port>:<external port>
	link:
		- <another container name>
```

*Note that the yaml file formats of different `docker compose` version vary.*

|              | Version 1                            | Version 2                                                    | Version 3 |
| ------------ | ------------------------------------ | ------------------------------------------------------------ | --------- |
| Network      | attached to default Network *Bridge* | create a new network by default; and automatically link each other by services name(so you don’t need to link manually) |           |
| new features |                                      | `depends_on`, `networks`                                     |           |

{% include img.html src="docker.assets/image-20200413232225517.png" alt="image-20200413232225517" %} 

{% include img.html src="docker.assets/image-20200413232555258.png" alt="image-20200413232555258" %} 

## Docker Registry

image identifier: `<registry name>/<user name>/<image name>`

​	- usual registry name: `docker.io`(default), `gcr.io`(google cloud registry)

```bash
# private-regristry
docker login <private-registry.io>
ducker run <private-rigsitry.io>/app/internal-app


# deploy private registry
docker run -d -p 5000:5000 --name registry registry:2
docker image tag <image name> localhost:5000/<image name>
docker push localhost:5000/<image name>
```

## Docker Engine

- components of docker engine
  - Docker CLI: could be run on another machine
  - REST API
  - Docker Deamon
- cgroups(control groups): limit the resources used by a containers

```bash
docker -H=<remote machine address with docker engine>:2375

docker run --cpus=.5 <image>
docker run --memory=100m <image>
```

## Docker on Windows

- two options to run Linux containers on windows OS(run on Linux VM)
  - docker toolbox: integrate *Oracle VirtualBox*
  - docker desktop for Windows: run on *Microsoft Hyper-V*
- Run Windows containers on windows OS
  - two strategies
    - shared kernel
    - Hyper-V isolation(self-own kernel)
  - Based Images
    - Windows Server Core
    - Nano Server

> VirtualBox or Hyper-V?
>
> - can’t co-exist!

## Docker Orchestration

- [Concept] Docker Orchestration: for the purpose of load balancing
- solutions: docker swarm, kubernetes(google, the most popular), mesos(paget)
### Docker Swarm’s Brief

- stack layout: container(applications, libs, deps) -> multiple docker hosts -> docker swamp -> kernel -> VM
- docker hosts: one is called swarm manager, others are called Node workers

```bash
# docker swarm
docker service create --replicas=100 <image name>
```

### Kubernetes’s Brief

- stack layout: containers -> Node -> Cluster(with one master node)