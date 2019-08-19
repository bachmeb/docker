# docker - get started

## References
* https://docs.docker.com/get-started/
* https://docs.docker.com/install/linux/docker-ce/centos/
* https://github.com/NaturalHistoryMuseum/scratchpads2/wiki/Install-Docker-and-Docker-Compose-(Centos-7)

### Inspect system
#### Check the system linux version
```
cat /etc/redhat-release 
```
```bash
# Red Hat Enterprise Linux Server release 7.3 (Maipo)
```
#### Check the system network adapters
```
ip addr show
```
```java
/*
+ 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:50:dd:b6:dd:b3 brd ff:ff:ff:ff:ff:ff
    inet 10.158.176.71/24 brd 10.158.176.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::345:56ff:feb6:9sdf/64 scope link 
       valid_lft forever preferred_lft forever
*/
```
#### Check system firewall
```
sudo iptables -L
[sudo] password for brianb: 
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (0 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION (0 references)
target     prot opt source               destination         

Chain FORWARD_IN_ZONES (0 references)
target     prot opt source               destination         

Chain FORWARD_IN_ZONES_SOURCE (0 references)
target     prot opt source               destination         

Chain FORWARD_OUT_ZONES (0 references)
target     prot opt source               destination         

Chain FORWARD_OUT_ZONES_SOURCE (0 references)
target     prot opt source               destination         

Chain FORWARD_direct (0 references)
target     prot opt source               destination         

Chain FWDI_public (0 references)
target     prot opt source               destination      

Chain FWDI_public_allow (0 references)
target     prot opt source               destination         

Chain FWDI_public_deny (0 references)
target     prot opt source               destination         

Chain FWDI_public_log (0 references)
target     prot opt source               destination         

Chain FWDO_public (0 references)
target     prot opt source               destination         

Chain FWDO_public_allow (0 references)
target     prot opt source               destination         

Chain FWDO_public_deny (0 references)
target     prot opt source               destination         

Chain FWDO_public_log (0 references)
target     prot opt source               destination         

Chain INPUT_ZONES (0 references)
target     prot opt source               destination         

Chain INPUT_ZONES_SOURCE (0 references)
target     prot opt source               destination         

Chain INPUT_direct (0 references)
target     prot opt source               destination         

Chain IN_public (0 references)
target     prot opt source               destination         

Chain IN_public_allow (0 references)
target     prot opt source               destination         

Chain IN_public_deny (0 references)
target     prot opt source               destination         

Chain IN_public_log (0 references)
target     prot opt source               destination         

Chain OUTPUT_direct (0 references)
target     prot opt source               destination         
```

### Set up your Docker environment
#### Install device mapper and lvm2
```
sudo yum install yum-utils device-mapper-persistent-data lvm2
```
#### Add the yum repo for docker-ce
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
#### Search yum for docker
```
sudo yum search docker
```
```bash
# docker.x86_64 : Automates deployment of containerized applications
```
#### Search yum for docker-ce
```
sudo yum search docker-ce
```
```bash
# docker-ce.x86_64 : The open-source application container engine
# docker-ce-cli.x86_64 : The open-source application container engine
# docker-ce-selinux.noarch : SELinux Policies for the open-source application container engine
```
#### Install docker-ce, docker-ce, and containerd.io
```
sudo yum install docker-ce
```
```java
/*
===================================================================================================================================================
 Package                         Arch                     Version                             Repository                                      Size
===================================================================================================================================================
Installing:
 docker-ce                       x86_64                   3:18.09.2-3.el7                     docker-ce-stable                                19 M
Installing for dependencies:
 containerd.io                   x86_64                   1.2.2-3.3.el7                       docker-ce-stable                                22 M
 docker-ce-cli                   x86_64                   1:18.09.2-3.el7                     docker-ce-stable                                14 M
 libtool-ltdl                    x86_64                   2.4.2-22.el7_3                      rhui-rhel-7-server-rhui-rpms                    49 k

Transaction Summary
===================================================================================================================================================
Install  1 Package (+3 Dependent packages)
*/
#### See that the docker group has been created
```
cat /etc/group | grep docker
```
```bash
# docker:x:498:
```
```
systemctl
```
```
sudo usermod -aG docker USERNAME
su USERNAME
```
```
sudo systemctl start docker
```
```
docker --version
```
```
sudo docker run hello-world
```
```java
/*
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
*/
```
```
docker info
```
```
docker image ls
```
```
docker container ls --all
```
```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```




### Build an image and run it as one container
```
cd ~
mkdir -p docker/get-started
cd docker/get-started
```
```
vi Dockerfile
```
```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
```
vi requirements.txt
```
```
Flask
Redis
```
```
vi app.py
```
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
```
ls
```
```
Dockerfile		app.py			requirements.txt
```
```
docker build --tag=friendlyhello .
```
```
docker image ls
```
```
docker run -p 4000:80 friendlyhello
```
Open a new SSH connection to the same machine
```
curl localhost:4000
```
```
<h3>Hello World!</h3><b>Hostname:</b> 981a9ef44656<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>[brian@ip-172-31-95-249 ~]$
```

### Scale your app to run multiple containers

### Distribute your app across a cluster

### Stack services by adding a backend database

### Deploy your app to production
