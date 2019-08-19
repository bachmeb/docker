# docker - jboss-eap-7.0.0
This guide uses an EC2 VM in AWS

## References
* https://docs.docker.com/get-started/
* https://hub.docker.com/r/zheiro/jdk8-jboss-eap7/

### Prepare
#### Download JBoss EAP 7.0.0
* https://developers.redhat.com/products/eap/download

#### Choose the zip file
* https://developers.redhat.com/download-manager/file/jboss-eap-7.0.0.zip

#### Keep a copy of the zip file on your local machine
```
~/Downloads
```

#### Create a new vm
* https://aws.amazon.com/ec2/

##### Step 1: Choose an Amazon Machine Image (AMI)
```
Amazon Linux 2 AMI (HVM), SSD Volume Type - ami-0b898040803850657 (64-bit x86) / ami-0ad82a384c06c911e (64-bit Arm)
```
##### Step 2: Choose an Instance Type
```
	Family: General purpose
	Type: t2.micro
	vCPUs: 1
	Memory: 1
	Instance Storage: EBS only
	EBS-Optimized: -
	Network Performance: Low to Moderate
```
##### Step 3: Configure Instance Details
```
	Number of instances: 	1
	Network: (default)
	Subnet: No preference (default subnet in any Availability Zone)
	Assign Public IP: Use subnet setting (Enable)
	IAM role: None
```

##### Step 4: Add Storage
```
Volume Type: Root
Device: /dev/xvda
Snapshot: snap-ad8e61f8
Size (GiB): 8
Volume Type: General Purpose SSD (GP2)
IOPS: 100 / 3000
Delete on Termination: Yes
Encrypted: Not Encrypted
```
##### Step 5: Tag Instance
Key: Name
Value: Docker VM with JBoss

##### Step 6: Configure Security Group 
*Allow all ICMP and TCP traffic from your IP address. SSH communicates via TCP on port 22.*
```
Type       Protocol   Port Range   Source
All TCP    TCP        0 - 65535    your ip address/32
All ICMP   All        N/A          your ip address/32
```
##### Download the pem file and change the mode to 400
```
chmod 400 pemfile.pem
```
##### Ping the Elastic IP address to confirm that ICMP traffic is allowed from your IP address
```
ping [ec2.ipa.ddr.ess]
```
##### Connect via SSH
```
ssh -i pemfile.pem ec2-user@[ec2.ipa.ddr.ess]
```
##### Check the Linux distro version
```
cat /proc/version
```
```
Linux version 4.14.123-111.109.amzn2.x86_64 (mockbuild@ip-10-0-1-12) (gcc version 7.3.1 20180303 (Red Hat 7.3.1-5) (GCC)) #1 SMP Mon Jun 10 19:37:57 UTC 2019
```

##### Install system updates
```
sudo yum update	
```

##### Check the time
```
date
```

##### List the available time zones
```
ls /usr/share/zoneinfo/
```

##### Update the /etc/sysconfig/clock file with your time zone
```
sudo vi /etc/sysconfig/clock
```
```
ZONE="America/New_York"
UTC=false
```
##### Create a symbolic link between /etc/localtime and your time zone file
```
sudo ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
```

##### Check the time
```
date
```

##### Reboot the system to pick up the new hostname and time zone information in all services and applications
```
sudo reboot
```

##### Copy the JBoss EAP zip file to the EC2 VM
```
scp -i pemfile.pem ~/Downloads/jboss-eap-7.0.0.zip ec2-user@[ec2.ipa.ddr.ess]:~
```

##### Connect via SSH
```
ssh -i pemfile.pem ec2-user@[ec2.ipa.ddr.ess]
```

##### Move the zip file to the /tmp directory
```
mv ~/jboss-eap-7.0.0.zip /tmp
```

##### Check the time
```
date
```
	
##### Check the default options configured for adding a user on this system
```
sudo useradd -D
```

##### Create a non-admin user account for yourself
```
sudo useradd [your user account name]
```

##### Set your password
```
sudo passwd [your user account name]
```

##### Add your account to the wheel group
```
sudo usermod -aG wheel [your user account name]
```

##### Switch to your user account
```
su -l [your user account name]
```
	
##### Go home
```
cd ~
pwd
```

##### Read your .bashrc file
```
cat .bashrc
```

##### Check the hostname of the ec2 instance
```
hostname
```

##### Check the DNS domain name of the ec2 instance
```
dnsdomainname
```

##### Get the IP address of the internal name server
```
cat /etc/resolv.conf
```

##### Check the routing table
```
route
```

##### Check the internal IP address
```
ifconfig
```

##### Check the external IP address
```
wget http://ipinfo.io/ip -qO -
```

##### Run a speed test (just for fun)
```
wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
chmod +x speedtest-cli
./speedtest_cli
```

#### Install OpenJDK 1.8
```
yum search openjdk
sudo yum install java-1.8.0-openjdk
```
```
java -version
```
```
openjdk version "1.8.0_201"
OpenJDK Runtime Environment (build 1.8.0_201-b09)
OpenJDK 64-Bit Server VM (build 25.201-b09, mixed mode)
```

#### Install Docker
```
sudo yum search docker
```
```
docker.x86_64 : Automates deployment of containerized applications
```
```
sudo yum install docker
```
```
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                                                                                              | 2.4 kB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package docker.x86_64 0:18.06.1ce-10.amzn2 will be installed
--> Processing Dependency: pigz for package: docker-18.06.1ce-10.amzn2.x86_64
--> Processing Dependency: libcgroup for package: docker-18.06.1ce-10.amzn2.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-18.06.1ce-10.amzn2.x86_64
--> Running transaction check
---> Package libcgroup.x86_64 0:0.41-15.amzn2 will be installed
---> Package libtool-ltdl.x86_64 0:2.4.2-22.2.amzn2.0.2 will be installed
---> Package pigz.x86_64 0:2.3.4-1.amzn2.0.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===============================================================================================================================================================================================
 Package                                     Arch                                  Version                                              Repository                                        Size
===============================================================================================================================================================================================
Installing:
 docker                                      x86_64                                18.06.1ce-10.amzn2                                   amzn2extra-docker                                 37 M
Installing for dependencies:
 libcgroup                                   x86_64                                0.41-15.amzn2                                        amzn2-core                                        65 k
 libtool-ltdl                                x86_64                                2.4.2-22.2.amzn2.0.2                                 amzn2-core                                        49 k
 pigz                                        x86_64                                2.3.4-1.amzn2.0.1                                    amzn2-core                                        81 k

Transaction Summary
===============================================================================================================================================================================================
Install  1 Package (+3 Dependent packages)

Total download size: 37 M
Installed size: 151 M
Is this ok [y/d/N]: y
```

```
cat /etc/group | grep docker
```

```bash
# docker:x:498:
systemctl
sudo usermod -aG docker USERNAME
su USERNAME
sudo systemctl start docker
docker --version
sudo docker run hello-world
```
```
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
```
```
docker info
docker image ls
docker container ls --all
```
## Build JBoss image
```
mkdir -p ~/docker/jboss/eap7
```
```
cd ~/docker/jboss/eap7/
```
```
vi Dockerfile
```
```
#Use latest jboss/base-jdk:8 image as the base
FROM jboss/base-jdk:8

#Set the JBOSS_VERSION env variable
ENV JBOSS_VERSION 7.0.0
ENV JBOSS_HOME /opt/jboss/jboss-eap-7.0/

#Copy jboss eap zip file into docker image
COPY jboss-eap-$JBOSS_VERSION.zip $JBOSS_HOME

# Unzip eap to Jboss home
RUN cd $JBOSS_HOME
&& unzip jboss-eap-$JBOSS_VERSION.zip
&& rm jboss-eap-$JBOSS_VERSION.zip

# Make jboss user owner of jboss home
chown -R $JBOSS_HOME jboss

# Ensure signals are forwarded to the JVM process correctly for graceful shutdown
ENV LAUNCH_JBOSS_IN_BACKGROUND true

# Run as the jboss user
USER jboss

# Expose port 8080 for JBoss
EXPOSE 8080

# Set the default command to run on boot
# This will boot JBoss EAP in the standalone mode and bind to all interface
CMD ["/opt/jboss/jboss-eap-7.0/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]
```
