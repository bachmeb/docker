# docker - get started

## References
* https://docs.docker.com/get-started/
* https://docs.docker.com/install/linux/docker-ce/centos/

### Check linux version
```
$ cat /etc/redhat-release 
```
```
# Red Hat Enterprise Linux Server release 7.3 (Maipo)
```

### Set up your Docker environment
```
sudo yum install yum-utils device-mapper-persistent-data lvm2
```
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
sudo yum search docker
```
```bash
# docker.x86_64 : Automates deployment of containerized applications
```
```
sudo yum install docker-ce
sudo yum install docker-ce-cli
sudo yum install containerd.io
```
```
cat /etc/group | grep docker
```
```
# docker:x:498:
```

### Build an image and run it as one container

### Scale your app to run multiple containers

### Distribute your app across a cluster

### Stack services by adding a backend database

### Deploy your app to production
