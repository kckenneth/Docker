# Docker Commands

This is about Docker commands 

---
title: "Docker"
author: "Kenneth Chen"
date: "7/2/2018"
output: pdf_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


# To check  

```
$ docker ps -a              # list containers, including the ones running in the background
$ docker-compose ps         # list cluster 
$ docker images             # list images
$ docker image ls  
$ docker container ls       # list containers, ~ $ docker ps -a
$ docker container ls -q
$ docker-machine ls         # list VM, virtual machine
$ docker node ls            # only when you have VM running
$ docker service ls
```

# To start

```
$ docker-compose up -d
```

# To end

```
$ docker container stop <ID>
$ docker-compose down
$ docker rm <ID>
$ docker rm -f $(docker ps -aq)       # force quit all containers
```

# Building A Docker Image 
## 1. I. Dockerfile + II. requirement.txt + III. app.py  

#### I. Dockerfile  

```
$ vi Dockerfile
```

Copy and paste the following dockerfile with all the instructions  

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Your container will be built upon the base image specified by **`FROM python:2.7-slim`**. When this container is initiated by **`docker run`**, the first step it will do is, run the **`app.py`** as follows

```
$ python app.py
```

So instead of we manually sending the command, once the container is launched, it will run the command. So intuitively, we need to provide the **`app.py`** script. Port `80` is in order for container to communicate outside world. If you have additional files you want to start off with, let's say you develop a web application that needs some template.html, about.html so on and so forth. So you need to have those files in the container. So you need to copy necessary file and build into the container. The following lines are optional, not required for this exercise. 

```
# install Python modules needed by the Python app
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# copy files required for the app to run
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/
```


#### II. requirement.txt  

```
$ vi requirement.txt  
```

Copy and paste the following. This shows that two libraries or packages are required and will be called upon docker container creation.  
```
Flask  
Redis  
```

#### III. app.py  

```
$ vi app.py  
```

copy and paste the following.  

```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

## 2. Docker build  

```
$ docker build -t <image name> <directory path to Dockerfile>
$ docker build -t friendlyhello .
```

## 3. Run the app

```
$ docker run -p 4000:80 <image name>  
$ docker run -d -p 4000:80 <image name>  
```

#### To Test
```
$ curl http://localhost:4000  
```

Note. The port. 

## 4. Docker login to save the docker image
```
$ docker login
```
## 5. Tag and Share the image
```
$ docker tag <image name> <user name>/<repository>:<tag>  
$ docker tag ken kckenneth/tutorial:part1  

$ docker push <user name>/<repository>:<tag>  
```

## 6. Docker-compose

```
$ vi docker-compose.yml  
```

Copy and paste the following cluster information

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

The image in the cluster will become

```
    image: kckenneth/tutorial:part1 
```

Note. The image will be created as **web** container in the cluster once it's instantiated.   

## 7. Docker Swarm 

This will initiate a swarm manager in your local machine  

```
$ docker swarm init
$ docker stack deploy -c docker-compose.yml <app name>  
$ docker stack deploy -c docker-compose.yml getstartedlab  
```

#### Take down the app and leave the swarm  

```
$ docker stack rm getstartedlab  
$ docker swarm leave --force  
```

You use --force when you are a swarm manager. Otherwise, you can leave the flag.  

## 8. Virtual Machine (VM) + Swarm Manager and Workers 

Create two VMs, one for the purpose of swarm manager VM, and the other for swarm worker purpose  

```
$ docker-machine create --driver virtualbox <VM_name>

$ docker-machine create --driver virtualbox myvm1  
$ docker-machine create --driver virtualbox myvm2  

$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip:port>"
```

This message will appear **Swarm initialized: current node <node ID> is now a manager.** 

You can now ssh into VM.

```
$ docker-machine ssh myvm1  
$ docker-machine ssh myvm2 

$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"  
```

You can also directly query  
```
$ docker-machine ssh myvm1 "docker node ls"  
$ docker
```


Dockerfile > (Build) > Image > (Run) > Container.

Dockerfile: contains a set of docker instructions that provisions your operating system the way you like, and installs/configure all your software's.

Image: compiled Dockerfile. Saves you time from rebuilding the Dockerfile every time you need to run a container. And it's a way to hide your provision code.

Container: the virtual operating system itself, you can ssh into it and run any commands you wish, as if it's a real environment. You can run 1000+ containers from the same Image.

# Docker Containerization 
### One line command
```
$ docker run ubuntu ls -l
total 64
drwxr-xr-x   2 root root 4096 Apr 26 21:17 bin
drwxr-xr-x   2 root root 4096 Apr 24 08:34 boot
drwxr-xr-x   5 root root  340 Jul 16 02:46 dev
drwxr-xr-x   1 root root 4096 Jul 16 02:46 etc
drwxr-xr-x   2 root root 4096 Apr 24 08:34 home
drwxr-xr-x   8 root root 4096 Apr 26 21:16 lib
...
...
```

What happened? Behind the scenes, a lot of stuff happened. When you call **`run`**,

- The Docker client contacts the Docker daemon  
- The Docker daemon checks local store if the image (**`ubuntu`** in this case) is available locally, and if not, downloads it from Docker Store.  
- The Docker daemon creates the container (image's instance) and then runs a command **`ls -l`** in that container.  
- The Docker daemon streams the output of the command to the Docker client.  

```
$ docker run ubuntu echo "I'm Kenneth."
I'm Kenneth.
```

**Note**  
You cannot run with a pipe or double ampersand &&. It'd run but the 2nd command is not for containers, rather the 2nd command is the host.  
```
$ docker run ubuntu && ls -l          # will list host files, not docker containers files
$ docker run ubuntu | ls -l
```
Although those simple commands can be run directly from **`docker run`**, other scripted commands requires the user to go into the shell. 

```
$ docker run -it ubuntu
```

# Inspecting the Docker image

```
$ docker inspect ubuntu
[
    {
        "Id": "sha256:452a96d81c30a1e426bc250428263ac9ca3f47c9bf086f876d11cb39cf57aeec",
        "RepoTags": [
            "ubuntu:latest"
        ],
        "RepoDigests": [
            "ubuntu@sha256:c8c275751219dadad8fa56b3ac41ca6cb22219ff117ca98fe82b42f24e1ba64e"
        ],
        "Parent": "",
        "Comment": "",
        ...
        ...
```



# Static Image 

```
$ docker run -d -p 4000:80 dockersamples/static-site
```

sha key pops up and you can go to 0.0.0.0:4000 to view the static image
```
b0ff4078722c2d99eff57ef6b8573ed338e33581fb9c9b333ffe399dc579b21c
```

## Docker Port or Not

When you run the docker image instances (container), those containers will run instantly and stop unless the container has some functionality that keeps running at the background. For eg, you have an image **`ubuntu`** and you create a container out of the image. 

```
$ docker run ubuntu
```

The container **`ubuntu`** will quickly run and stop. It's because you didn't add any commands in the `docker run`. Of course, ubuntu is one of the Linux OS and you can play around in the ubuntu container. The fact that you'd be staying in the Ubuntu environment a little longer means that you'll be interacting with the Ubuntu environment. So you'd call the additional command `it = interactive`. 

```
$ docker run -it ubuntu

root@faf19747658b:/#
```

You're now in the Ubuntu environment by virtue of docker `ubuntu` container. You can now uses some commands such as `ls, pwd, whoami` etc in the ubuntu environment. Imagine some containers have web applications that you can communicate via web browser. The fact that they have web applications means that they must have established port in order to communicate outside of the container. So when you want to run those docker images, you can run the container by specifying the port number. 

```
$ docker run -d -p 4000:80 dockersamples/static-site
```

This will run the docker image **`dockersamples/static-site`**. You also specify the port 4000 will be open. So you can now go to the web browser and check if the web applications is running. 

```
http://0.0.0.0:4000
```

Or you can also check the web application in CLI  

```
$ curl http://0.0.0.0:4000
```

This will display all the messages of the web applications. Even if you forget which port you specify, you can call up by

```
$ docker ps
```

This will list all the containers running with their status. You can also call the port by  

```
$ docker port <container ID>
$ docker port f54569eee2e2

80/tcp -> 0.0.0.0:4000
```

It says the container is communicating with your host machine (imagine that container is another machine) through port 4000.  


# Docker images

When you bash into container, you're basically inside the container with a bash command. 
```
$ docker run -it midsw205/base:latest bash
```
This command initiate docker container with `docker run`, and calling interactive `it` followed by the image name: `midsw205/base:latest`. The last command `bash` tells that you're going inside the container and will communicate in bash mode. 

Imagine if your docker image offer something else, some functions that you'd like to explore, eg, spark, python, you can just call the images and call the function of the app following the command. 

```
$ docker run -it midsw205/spark-python:0.0.5 pyspark
```

If you want to mount the volume
```
$ docker run -it -v /Users/test:/test midsw205/spark-python:0.0.5 pyspark
```
