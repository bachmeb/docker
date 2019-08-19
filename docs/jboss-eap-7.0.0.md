# docker - jboss-eap-7.0.0
This guide uses an EC2 VM in AWS

## References
* https://docs.docker.com/get-started/
* https://hub.docker.com/r/zheiro/jdk8-jboss-eap7/
* http://blog.adeel.io/2016/11/06/amazon-ec2-ssh-timeout-due-to-inactivity/

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

The options selected in creating this VM are for a proof-of-concept build only. This VM is intended to be used in the context of following a tutorial, not as a production system. 

##### Step 1: Choose an Amazon Machine Image (AMI)
```
Amazon Linux 2 AMI (HVM), SSD Volume Type - ami-0b898040803850657 (64-bit x86) / ami-0ad82a384c06c911e (64-bit Arm)
```
##### Step 2: Choose an Instance Type
```
Family: General purpose
Type: t2.medium
vCPUs: 1
Memory: 4
Instance Storage: EBS only
EBS-Optimized Available: Yes
Network Performance: Low to Moderate
```
##### Step 3: Configure Instance Details
```
Number of instances: 1
Purchasing option: unchecked
Network: (default)
Subnet: No preference (default subnet in any Availability Zone)
Assign Public IP: Use subnet setting (Enable)
Placement group: unchecked
Capacity Reservation: Open
IAM role: None
Shutdown behavior: Stop
Enable termination protection: NO
Monitoring: NO
Tenancy: Shared
Elastic Inference: NO
T2/T3 Unlimited: NO
```

##### Step 4: Add Storage
```
Volume Type: Root
Device: /dev/xvda
Snapshot: snap-ad8e61f8
Size (GiB): 32
Volume Type: General Purpose SSD (GP2)
IOPS: 100 / 3000
Throughput (MB/s): N/A
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
##### Edit the ssh config file on your local machine to prevent AWS EC2 SSH timeout every 60 seconds
```
vi ~/.ssh/config
```
```
ServerAliveInterval 50
```
##### Connect via SSH
```
ssh -v -i pemfile.pem ec2-user@[ec2.ipa.ddr.ess]
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

##### Copy the JBoss EAP zip file to the /tmp directory on the EC2 VM
```
scp -i pemfile.pem ~/Downloads/jboss-eap-7.0.0.zip ec2-user@[ec2.ipa.ddr.ess]:/tmp
```

##### Connect via SSH
```
ssh -i pemfile.pem ec2-user@[ec2.ipa.ddr.ess]
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
su [your user account name]
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
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.42.14  netmask 255.255.240.0  broadcast 172.31.47.255
        inet6 fe80::ca3:bcff:fe89:c6ca  prefixlen 64  scopeid 0x20<link>
        ether 0e:a3:bc:89:c6:ca  txqueuelen 1000  (Ethernet)
        RX packets 492619  bytes 684798197 (653.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 105220  bytes 151237789 (144.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 10  bytes 2373 (2.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10  bytes 2373 (2.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

##### Check the external IP address
```
wget http://ipinfo.io/ip -qO -
```

##### Run a speed test (just for fun)
```
wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
chmod +x speedtest-cli
./speedtest-cli
```
```
Retrieving speedtest.net configuration...
Testing from Amazon (ip address)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Shentel Service Company (Ashburn, VA) [0.98 km]: 3.861 ms
Testing download speed................................................................................
Download: 883.80 Mbit/s
Testing upload speed................................................................................................
Upload: 854.43 Mbit/s
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
#### Install JBoss
Running JBoss on the EC2 VM shows that the VM has enough memory to run JBoss at all...
```
cd /tmp	
unzip jboss-eap-7.0.0.zip
cd jboss-eap-7.0.0
bin/standalone.sh
```
```
=========================================================================

  JBoss Bootstrap Environment

  JBOSS_HOME: /tmp/jboss-eap-7.0

  JAVA: java

  JAVA_OPTS:  -server -verbose:gc -Xloggc:"/tmp/jboss-eap-7.0/standalone/log/gc.log" -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms1303m -Xmx1303m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true

=========================================================================

11:15:42,737 INFO  [org.jboss.modules] (main) JBoss Modules version 1.5.1.Final-redhat-1
11:15:42,951 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.6.Final-redhat-1
11:15:43,032 INFO  [org.jboss.as] (MSC service thread 1-3) WFLYSRV0049: JBoss EAP 7.0.0.GA (WildFly Core 2.1.2.Final-redhat-1) starting
11:15:44,354 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0039: Creating http management service using socket-binding (management-http)
11:15:44,379 INFO  [org.xnio] (MSC service thread 1-4) XNIO version 3.3.6.Final-redhat-1
11:15:44,453 INFO  [org.xnio.nio] (MSC service thread 1-4) XNIO NIO Implementation Version 3.3.6.Final-redhat-1
11:15:44,528 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 37) WFLYIO001: Worker 'default' has auto-configured to 4 core threads with 32 task threads based on your 2 available processors
11:15:44,539 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 38) WFLYCLINF0001: Activating Infinispan subsystem.
11:15:44,541 INFO  [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool -- 33) WFLYJCA0004: Deploying JDBC-compliant driver class org.h2.Driver (version 1.3)
11:15:44,593 INFO  [org.jboss.remoting] (MSC service thread 1-4) JBoss Remoting version 4.0.18.Final-redhat-1
11:15:44,595 INFO  [org.jboss.as.connector] (MSC service thread 1-1) WFLYJCA0009: Starting JCA Subsystem (WildFly/IronJacamar 1.3.3.Final-redhat-1)
11:15:44,603 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-1) WFLYJCA0018: Started Driver service with driver-name = h2
11:15:44,617 INFO  [org.jboss.as.jsf] (ServerService Thread Pool -- 44) WFLYJSF0007: Activated the following JSF Implementations: [main]
11:15:44,678 INFO  [org.jboss.as.naming] (ServerService Thread Pool -- 46) WFLYNAM0001: Activating Naming Subsystem
11:15:44,743 WARN  [org.jboss.as.txn] (ServerService Thread Pool -- 54) WFLYTX0013: Node identifier property is set to the default value. Please make sure it is unique.
11:15:44,763 INFO  [org.jboss.as.webservices] (ServerService Thread Pool -- 56) WFLYWS0002: Activating WebServices Extension
11:15:44,771 INFO  [org.jboss.as.security] (ServerService Thread Pool -- 53) WFLYSEC0002: Activating Security Subsystem
11:15:44,780 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) WFLYUT0003: Undertow 1.3.21.Final-redhat-1 starting
11:15:44,787 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 55) WFLYUT0003: Undertow 1.3.21.Final-redhat-1 starting
11:15:44,792 INFO  [org.jboss.as.security] (MSC service thread 1-4) WFLYSEC0001: Current PicketBox version=4.9.6.Final-redhat-1
11:15:44,824 INFO  [org.jboss.as.naming] (MSC service thread 1-2) WFLYNAM0003: Starting Naming Service
11:15:44,825 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-3) WFLYMAIL0001: Bound mail session [java:jboss/mail/Default]
11:15:44,987 INFO  [org.jboss.as.ejb3] (MSC service thread 1-1) WFLYEJB0482: Strict pool mdb-strict-max-pool is using a max instance size of 8 (per class), which is derived from the number of CPUs on this host.
11:15:44,987 INFO  [org.jboss.as.ejb3] (MSC service thread 1-4) WFLYEJB0481: Strict pool slsb-strict-max-pool is using a max instance size of 32 (per class), which is derived from thread worker pool sizing.
11:15:45,048 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 55) WFLYUT0014: Creating file handler for path '/tmp/jboss-eap-7.0/welcome-content' with options [directory-listing: 'false', follow-symlink: 'false', case-sensitive: 'true', safe-symlink-paths: '[]']
11:15:45,064 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) WFLYUT0012: Started server default-server.
11:15:45,091 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-4) WFLYUT0018: Host default-host starting
11:15:45,244 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) WFLYUT0006: Undertow HTTP listener default listening on 127.0.0.1:8080
11:15:45,624 INFO  [org.infinispan.factories.GlobalComponentRegistry] (MSC service thread 1-1) ISPN000128: Infinispan version: Infinispan 'Mahou' 8.1.2.Final-redhat-1
11:15:45,644 INFO  [org.jboss.ws.common.management] (MSC service thread 1-3) JBWS022052: Starting JBossWS 5.1.3.SP1-redhat-1 (Apache CXF 3.1.4.redhat-1)
11:15:45,652 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-1) WFLYDS0013: Started FileSystemDeploymentService for directory /tmp/jboss-eap-7.0/standalone/deployments
11:15:45,726 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-3) WFLYJCA0001: Bound data source [java:jboss/datasources/ExampleDS]
11:15:45,882 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
11:15:45,890 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
11:15:45,890 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: JBoss EAP 7.0.0.GA (WildFly Core 2.1.2.Final-redhat-1) started in 3475ms - Started 267 of 553 services (371 services are lazy, passive or on-demand)
```
Hit ctrl-c to stop JBoss
##### Run JBoss in the background
```
nohup bin/standalone.sh &
```
```
curl localhost:8080
```
```
<!DOCTYPE html>
<html>
<head>

    <title>JBoss EAP 7</title>
    <!-- proper charset -->
    <meta http-equiv="content-type" content="text/html;charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE8" />
    <link rel="stylesheet" type="text/css" href="eap.css" />
    <link rel="shortcut icon" href="favicon.ico" />
</head>

<body>

<div id="container" style="position: absolute; left: 0px; top: 0px; right: 0px; bottom: 0px;">

  <!-- header -->
  <div class="header-panel">
    <div class="header-line">&nbsp;</div>
    <div class="header-top">
      <img class="prod-title" src="images/product_title.png"/><span class="prod-version">7</span>
    </div>
    <div class="header-bottom">&nbsp;</div>
  </div>


  <!-- main content -->
  <div id="content">

    <div class="section">

      <h1>Welcome to JBoss EAP 7</h1>

      <h3>Your Red Hat JBoss Enterprise Application Platform is running.</h3>

      <p>
        <a href="/console">Administration Console</a> |
        <a href="https://access.redhat.com/documentation/en/jboss-enterprise-application-platform/">Documentation</a> |
        <a href="https://access.redhat.com/discussions">Online User Groups</a> <br/>
      </p>

      <sub>To replace this page simply deploy your own war with / as its context path.<br/>
        To disable it, remove the "welcome-content" handler for location / in the undertow subsystem.</sub>

    </div>

  </div>


  <div id="footer">&nbsp;</div>

</div>

</body >
</html>
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
#### Check the docker version
```
docker --version
```
#### Compare the network adapters with 
```
ip addr show
```
#### Check the firewall rules
```
sudo iptables -L
```
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
#### Check that the docker group was created
```
cat /etc/group | grep docker
```
```
docker:x:498:
```
##### Add your user account to the docker group
```
sudo usermod -aG docker USERNAME
su USERNAME
```
##### Start the docker daemon
```
systemctl
sudo systemctl start docker
```
#### Run the docker hello-world image
```
docker run hello-world
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
mkdir -p ~/docker/jboss/eap7a
cd ~/docker/jboss/eap7a/
pwd
```
```
vi Dockerfile
```
```
#Use latest jboss/base-jdk:8 image as the base
FROM jboss/base-jdk:8
```
#### Build the image
```
ll
docker build --tag=eap7a .
```
#### List the images
```
docker image ls
docker image ls -a
```
#### List the running containers
```
docker container ls
docker container ls -a
```

#### Run the container with an interactive shell
```
docker run -it -p 8080:8080 eap7a
```
#### Print the working directory
```
pwd
```
```
/opt/jboss
```
#### List the environment variables
The HOME directory for the jboss user in this image is set to /opt/jboss
```
env
```
```
HOSTNAME=e8638eaf991e
TERM=xterm
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/opt/jboss
JAVA_HOME=/usr/lib/jvm/java
SHLVL=1
HOME=/opt/jboss
_=/usr/bin/env
```
#### The /opt/jboss directory is empty
```
ll
```
```
total 0
```
#### Check the java version
```
java -version
```
```
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```
#### Exit
```
exit
```
#### See that the eap7a container has stopped
```
 docker container ls -a
```
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
e8638eaf991e        eap7a               "/bin/bash"         5 minutes ago       Exited (0) 19 seconds ago                          amazing_bell
1f55327b2e4a        eap7a               "/bin/bash"         6 minutes ago       Exited (0) 6 minutes ago                           competent_keller
eca553d5552e        hello-world         "/hello"            8 minutes ago       Exited (0) 8 minutes ago                           frosty_lamarr
3b66a90de26f        hello-world         "/hello"            About an hour ago   Exited (0) About an hour ago                       eloquent_colden
```
#### Remove all containers
```
docker container rm $(docker container ls -a -q)
```
#### Remove all images
```
docker rmi $(docker images -a -q)
```
#### Make a new image using the jboss eap zip file
```
mkdir -p ~/docker/jboss/eap7b
cd ~/docker/jboss/eap7b/
pwd
cp /tmp/jboss-eap-7.0.0.zip .
```
```
vi Dockerfile
```
```
#Use latest jboss/base-jdk:8 image as the base
FROM jboss/base-jdk:8

#Copy jboss eap zip file into docker image
COPY jboss-eap-7.0.0.zip $HOME
```
#### Build the image
```
ll
docker build --tag=eap7b .
```
#### List the images
```
docker image ls -a
```
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
eap7b               latest              194a8ed5bf98        19 seconds ago      677MB
jboss/base-jdk      8                   8a832b9f0f57        6 months ago        513MB
```
#### List the running containers
```
docker container ls -a
```
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
```
#### Run the container with an interactive shell
```
docker run -it -p 8080:8080 eap7b
```
```
whoami
```
```
jboss
```
```
pwd
```
```
/opt/jboss
```
```
ll
```
```
total 160136
-rw-r--r-- 1 root root 163977055 Aug 19 17:19 jboss-eap-7.0.0.zip
```
```
unzip jboss-eap-7.0.0.zip
```
```
Archive:  jboss-eap-7.0.0.zip
   creating: jboss-eap-7.0/
   creating: jboss-eap-7.0/modules/
   creating: jboss-eap-7.0/modules/system/
   creating: jboss-eap-7.0/modules/system/layers/
   creating: jboss-eap-7.0/modules/system/layers/base/
   creating: jboss-eap-7.0/modules/system/layers/base/ch/
   creating: jboss-eap-7.0/modules/system/layers/base/ch/qos/
   creating: jboss-eap-7.0/modules/system/layers/base/ch/qos/cal10n/
   creating: jboss-eap-7.0/modules/system/layers/base/ch/qos/cal10n/main/
   creating: jboss-eap-7.0/modules/system/layers/base/net/
   creating: jboss-eap-7.0/modules/system/layers/base/net/jcip/
   creating: jboss-eap-7.0/modules/system/layers/base/net/jcip/main/
   creating: jboss-eap-7.0/modules/system/layers/base/io/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/servlet/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/servlet/main/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/jsp/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/jsp/main/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/core/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/core/main/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/js/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/js/main/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/websocket/
   creating: jboss-eap-7.0/modules/system/layers/base/io/undertow/websocket/main/
   creating: jboss-eap-7.0/modules/system/layers/base/io/netty/
   creating: jboss-eap-7.0/modules/system/layers/base/io/netty/main/
   creating: jboss-eap-7.0/modules/system/layers/base/ibm/
   creating: jboss-eap-7.0/modules/system/layers/base/ibm/jdk/
   creating: jboss-eap-7.0/modules/system/layers/base/ibm/jdk/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/
   creating: jboss-eap-7.0/modules/system/layers/base/org/bouncycastle/
   creating: jboss-eap-7.0/modules/system/layers/base/org/bouncycastle/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/fusesource/
   creating: jboss-eap-7.0/modules/system/layers/base/org/fusesource/jansi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/fusesource/jansi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/client/hotrod/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/client/hotrod/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/jdbc/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/jdbc/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/remote/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/remote/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/commons/
   creating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/commons/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/json/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/json/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/enterprise/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/enterprise/concurrent/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/enterprise/concurrent/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/el/
   creating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/el/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/mod_cluster/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/mod_cluster/undertow/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/mod_cluster/undertow/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/manager/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/manager/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/elytron/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/elytron/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/naming/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/naming/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/jboss/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/jboss/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/infinispan/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/infinispan/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/singleton/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/singleton/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/service/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/service/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/undertow/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/undertow/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/server/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/server/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/iiop-openjdk/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/iiop-openjdk/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/embedded/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/embedded/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/mod_cluster/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/mod_cluster/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/security/manager/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/security/manager/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/io/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/io/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/clustering/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/clustering/singleton/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/clustering/singleton/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/rts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/rts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/request-controller/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/request-controller/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/bean-validation/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/bean-validation/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/messaging-activemq/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/messaging-activemq/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/jberet/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/jberet/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/picketlink/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/picketlink/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/undertow/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/undertow/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/jberet/
   creating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/jberet/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/yaml/
   creating: jboss-eap-7.0/modules/system/layers/base/org/yaml/snakeyaml/
   creating: jboss-eap-7.0/modules/system/layers/base/org/yaml/snakeyaml/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/scannotation/
   creating: jboss-eap-7.0/modules/system/layers/base/org/scannotation/scannotation/
   creating: jboss-eap-7.0/modules/system/layers/base/org/scannotation/scannotation/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/
   creating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-core-asl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-core-asl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-jaxrs/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-jaxrs/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-xc/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-xc/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-mapper-asl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-mapper-asl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jettison/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jettison/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/woodstox/
   creating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/woodstox/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/azure/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/azure/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/ext/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/ext/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/jcl-over-slf4j/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/jcl-over-slf4j/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/impl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/impl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/config/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/config/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/schema/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/schema/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/bindings/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/bindings/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate4-3/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate4-3/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/4.3/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/cdi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/cdi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/envers/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/envers/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/engine/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/engine/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/serialization-avro/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/serialization-avro/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jms/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jms/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/orm/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/orm/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jgroups/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jgroups/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/commons-annotations/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/commons-annotations/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate5/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate5/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/4.1/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/5.0/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jdom/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jdom/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketbox/
   creating: jboss-eap-7.0/modules/system/layers/base/org/picketbox/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/antlr/
   creating: jboss-eap-7.0/modules/system/layers/base/org/antlr/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/client/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/joda/
   creating: jboss-eap-7.0/modules/system/layers/base/org/joda/time/
   creating: jboss-eap-7.0/modules/system/layers/base/org/joda/time/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/dom4j/
   creating: jboss-eap-7.0/modules/system/layers/base/org/dom4j/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/omg/
   creating: jboss-eap-7.0/modules/system/layers/base/org/omg/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/omg/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/modules/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/modules/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/stdio/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/stdio/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/log4j/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/log4j/logmanager/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/log4j/logmanager/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/container/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/container/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/container/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/core/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/core/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/river/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/river/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/invocation/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/invocation/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/xacml/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/xacml/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jandex/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jandex/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remote-naming/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remote-naming/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/deployers/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/deployers/jboss-service-deployer/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/deployers/jboss-service-deployer/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/aesh/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/aesh/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/integration/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/integration/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/vfs/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/vfs/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb-client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb-client/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ear/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ear/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/web/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/web/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/appclient/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/appclient/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ejb/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ejb/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/common-beans/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/common-beans/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting-jmx/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting-jmx/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/jdbcadapters/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/jdbcadapters/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/genericjms/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/genericjms/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/genericjms/main/META-INF/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/sasl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/sasl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logmanager/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logmanager/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/nio/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/nio/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/netty/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/netty/netty-xnio-transport/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/netty/netty-xnio-transport/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jboss-transaction-spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jboss-transaction-spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson2-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson2-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/jose-jwt/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/jose-jwt/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-yaml-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-yaml-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxrs/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxrs/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxb-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxb-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-cdi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-cdi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-json-p-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-json-p-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-crypto/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-crypto/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-spring/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-spring/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-spring/main/bundled/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-spring/main/bundled/resteasy-spring-jar/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-multipart-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-multipart-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-atom-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-atom-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jettison-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jettison-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jsapi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jsapi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-validator-provider-11/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-validator-provider-11/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/iiop-client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/iiop-client/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/jul-to-slf4j-stub/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/jul-to-slf4j-stub/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/compensations/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/compensations/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/txframework/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/txframework/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/dmr/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/dmr/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/classfilewriter/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/classfilewriter/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/probe/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/probe/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/core/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/core/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/integration/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/integration/ext-content/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/integration/ext-content/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/integration/ext-content/main/bundled/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/threads/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/threads/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/msc/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/msc/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jacorb/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jacorb/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/network/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/network/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsr77/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsr77/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-scanner/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-scanner/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/mail/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/mail/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/standalone/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/standalone/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/pojo/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/pojo/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/naming/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/naming/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-add-user/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-add-user/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/version/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/version/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-interface/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-interface/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller-client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller-client/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-repository/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-repository/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxr/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxr/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/hibernate/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/hibernate/4/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/openjpa/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/openjpa/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/jgroups/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/jgroups/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/web/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/web/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/web/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/ejb3/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/ejb3/infinispan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/ejb3/infinispan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/platform-mbean/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/platform-mbean/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/sar/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/sar/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/management-client-content/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/management-client-content/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/configadmin/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/configadmin/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/system-jmx/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/system-jmx/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/eap/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/eap/dir/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/eap/dir/META-INF/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/cli/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/cli/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/server/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/server/integration/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/server/integration/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jdr/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jdr/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jdr/main/resources/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxrs/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxrs/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf-injection/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf-injection/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web-common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web-common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/transactions/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/transactions/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/logging/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/logging/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jmx/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jmx/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ee/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ee/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/self-contained/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/self-contained/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/appclient/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/appclient/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/console/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/console/eap/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/vault-tool/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/vault-tool/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/protocol/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/protocol/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/host-controller/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/host-controller/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/weld/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/weld/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-management/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-management/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/server/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/server/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/threads/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/threads/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/images/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/connector/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/connector/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/xts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/xts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/messaging/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/messaging/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/modcluster/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/modcluster/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ejb3/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ejb3/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ejb3/main/timers/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/process-controller/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/process-controller/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security-api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security-api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security-api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security-api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cmp/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cmp/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cli/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cli/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/remoting/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/remoting/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jaxbintros/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jaxbintros/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb/remote/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb/remote/protocol/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb/remote/protocol/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb3/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb3/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/saaj-impl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/saaj-impl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-server/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-server/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-client/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-undertow/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-undertow/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-udp/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-udp/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/sts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/sts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-factories/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-factories/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/wsprovide/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/wsprovide/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/wsconsume/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/wsconsume/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/api/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-client/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-client/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/common/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/common/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/spi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/spi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-undertow-httpspi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-undertow-httpspi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/staxmapper/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/staxmapper/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting3/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting3/remoting-jmx/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting3/remoting-jmx/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting3/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/avro/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/avro/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/log4j/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/log4j/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/xml-resolver/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/xml-resolver/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/xalan/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/xalan/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/ws-security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/ws-security/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/services-sts/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/services-sts/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/velocity/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/velocity/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/xerces/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/xerces/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/neethi/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/neethi/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/santuario/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/santuario/xmlsec/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/santuario/xmlsec/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/codec/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/codec/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/pool/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/pool/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/lang/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/lang/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/io/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/io/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/collections/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/collections/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/logging/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/logging/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/beanutils/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/beanutils/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/cli/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/cli/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/openjpa/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/openjpa/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/internal/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/internal/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/james/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/james/mime4j/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/james/mime4j/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/lib/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/lib/linux-i686/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/lib/linux-x86_64/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/protocol/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/protocol/hornetq/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/protocol/hornetq/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/ra/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/ra/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/xmlschema/
   creating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/xmlschema/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/
   creating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/jdt/
   creating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/jdt/ecj/
   creating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/jdt/ecj/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/persistence/
   creating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/persistence/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jsoup/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jsoup/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/javassist/
   creating: jboss-eap-7.0/modules/system/layers/base/org/javassist/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jberet/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jberet/jberet-core/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jberet/jberet-core/main/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jaxen/
   creating: jboss-eap-7.0/modules/system/layers/base/org/jaxen/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javaee/
   creating: jboss-eap-7.0/modules/system/layers/base/javaee/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javaee/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/nu/
   creating: jboss-eap-7.0/modules/system/layers/base/nu/xom/
   creating: jboss-eap-7.0/modules/system/layers/base/nu/xom/main/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/main/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/main/service-loader-resources/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/main/service-loader-resources/META-INF/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/main/service-loader-resources/META-INF/services/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/main/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/main/service-loader-resources/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/main/service-loader-resources/META-INF/
   creating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/main/service-loader-resources/META-INF/services/
   creating: jboss-eap-7.0/modules/system/layers/base/gnu/
   creating: jboss-eap-7.0/modules/system/layers/base/gnu/getopt/
   creating: jboss-eap-7.0/modules/system/layers/base/gnu/getopt/main/
   creating: jboss-eap-7.0/modules/system/layers/base/asm/
   creating: jboss-eap-7.0/modules/system/layers/base/asm/asm/
   creating: jboss-eap-7.0/modules/system/layers/base/asm/asm/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/json/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/json/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/json/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/sql/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/sql/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/sql/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/interceptor/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/interceptor/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/interceptor/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jsp/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jsp/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jsp/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jstl/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jstl/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jstl/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/mail/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/mail/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/mail/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/auth/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/auth/message/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/auth/message/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/auth/message/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/jacc/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/jacc/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/security/jacc/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/wsdl4j/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/wsdl4j/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/wsdl4j/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/rmi/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/rmi/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/rmi/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/resource/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/resource/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/resource/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/orb/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/orb/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/orb/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/transaction/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/transaction/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/transaction/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/batch/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/batch/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/batch/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/jms/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/jms/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/jms/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/faces/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/faces/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/faces/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/jws/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/jws/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/jws/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/rpc/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/rpc/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/rpc/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/soap/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/soap/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/soap/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/stream/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/stream/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/stream/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/bind/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/bind/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/bind/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/ws/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/ws/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/xml/ws/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/management/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/management/j2ee/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/management/j2ee/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/management/j2ee/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/validation/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/validation/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/validation/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/concurrent/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/concurrent/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/concurrent/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/annotation/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/annotation/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/annotation/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/activation/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/activation/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/activation/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/el/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/el/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/el/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ejb/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ejb/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ejb/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/inject/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/inject/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/inject/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ws/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ws/rs/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ws/rs/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/ws/rs/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/persistence/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/persistence/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/persistence/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/websocket/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/websocket/api/
   creating: jboss-eap-7.0/modules/system/layers/base/javax/websocket/api/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/
   creating: jboss-eap-7.0/modules/system/layers/base/com/github/
   creating: jboss-eap-7.0/modules/system/layers/base/com/github/spullara/
   creating: jboss-eap-7.0/modules/system/layers/base/com/github/spullara/mustache/
   creating: jboss-eap-7.0/modules/system/layers/base/com/github/spullara/mustache/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/github/relaxng/
   creating: jboss-eap-7.0/modules/system/layers/base/com/github/relaxng/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/classmate/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/classmate/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-core/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-core/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-databind/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-databind/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-annotations/
   creating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-annotations/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/google/
   creating: jboss-eap-7.0/modules/system/layers/base/com/google/guava/
   creating: jboss-eap-7.0/modules/system/layers/base/com/google/guava/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/microsoft/
   creating: jboss-eap-7.0/modules/system/layers/base/com/microsoft/azure/
   creating: jboss-eap-7.0/modules/system/layers/base/com/microsoft/azure/storage/
   creating: jboss-eap-7.0/modules/system/layers/base/com/microsoft/azure/storage/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/h2database/
   creating: jboss-eap-7.0/modules/system/layers/base/com/h2database/h2/
   creating: jboss-eap-7.0/modules/system/layers/base/com/h2database/h2/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/jsf-impl/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/jsf-impl/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/fastinfoset/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/fastinfoset/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/txw2/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/txw2/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/messaging/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/messaging/saaj/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/messaging/saaj/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/istack/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/istack/main/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xsom/
   creating: jboss-eap-7.0/modules/system/layers/base/com/sun/xsom/main/
   creating: jboss-eap-7.0/standalone/
   creating: jboss-eap-7.0/standalone/configuration/
   creating: jboss-eap-7.0/standalone/tmp/
   creating: jboss-eap-7.0/standalone/tmp/auth/
   creating: jboss-eap-7.0/standalone/lib/
   creating: jboss-eap-7.0/standalone/lib/ext/
   creating: jboss-eap-7.0/standalone/deployments/
   creating: jboss-eap-7.0/domain/
   creating: jboss-eap-7.0/domain/configuration/
   creating: jboss-eap-7.0/domain/tmp/
   creating: jboss-eap-7.0/domain/tmp/auth/
   creating: jboss-eap-7.0/domain/data/
   creating: jboss-eap-7.0/domain/data/content/
   creating: jboss-eap-7.0/welcome-content/
   creating: jboss-eap-7.0/welcome-content/images/
   creating: jboss-eap-7.0/welcome-content/fonts/
   creating: jboss-eap-7.0/bin/
   creating: jboss-eap-7.0/bin/init.d/
   creating: jboss-eap-7.0/bin/client/
   creating: jboss-eap-7.0/appclient/
   creating: jboss-eap-7.0/appclient/configuration/
   creating: jboss-eap-7.0/docs/
   creating: jboss-eap-7.0/docs/examples/
   creating: jboss-eap-7.0/docs/examples/configs/
   creating: jboss-eap-7.0/docs/schema/
   creating: jboss-eap-7.0/docs/licenses/
   creating: jboss-eap-7.0/.installation/
  inflating: jboss-eap-7.0/modules/system/layers/base/net/jcip/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/commons/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/elytron/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/singleton/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/service/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/spi/main/wildfly-clustering-web-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-soap-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate4-3/main/jipijapa-hibernate4-3-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/orm/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jgroups/main/hibernate-search-backend-jgroups-5.5.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/4.1/jipijapa-hibernate4-1-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/core/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/main/jboss-marshalling-1.4.10.Final-redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/web/main/jboss-metadata-web-10.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/network/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsr77/main/wildfly-jsr77-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-repository/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxr/main/wildfly-jaxr-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/spi/main/jipijapa-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/web/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/ejb3/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/platform-mbean/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/sar/main/wildfly-sar-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf-injection/main/weld-core-jsf-2.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/transactions/main/wildfly-transactions-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-ExtraBold.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xalan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xalan/main/serializer-2.7.1.redhat-10.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-security-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-bindings-soap-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/lang/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/io/main/commons-io-2.4.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/httpclient-4.5.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/rngom-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-Regular.ttf
  inflating: jboss-eap-7.0/docs/schema/jboss-as-sar_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-resource-adapters_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-remoting_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-service_7_0.xsd
  inflating: jboss-eap-7.0/docs/schema/application_5.xsd
  inflating: jboss-eap-7.0/docs/schema/xml.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ee_4_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-rts_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-config_4_0.xsd
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/servlet/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/json/main/javax.json-1.0.3.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/embedded/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/mod_cluster/main/wildfly-mod_cluster-extension-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-xc/main/jackson-xc-1.9.13.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/woodstox/main/woodstox-core-asl-4.4.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/integration/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/vfs/main/jboss-vfs-3.2.11.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxrs/main/resteasy-jaxrs-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/lucene-core-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/websocket/main/undertow-websockets-jsr-1.3.21.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/cryptacular-1.0.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate4-3/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/infinispan/main/hibernate-infinispan-5.0.9.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jdom/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketbox/main/picketbox-commons-1.0.0.final-redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/antlr/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/client/main/hornetq-core-client-2.4.7.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-yaml-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxrs/main/async-http-servlet-3.0-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-cdi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-json-p-provider/main/resteasy-json-p-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-json-p-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson-provider/main/resteasy-jackson-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-spring/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-multipart-provider/main/resteasy-multipart-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/probe/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/spi/main/weld-spi-2.3.0.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-add-user/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/version/main/wildfly-version-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-interface/main/wildfly-domain-http-interface-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/management-client-content/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/configadmin/main/wildfly-configadmin-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/configadmin/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/system-jmx/main/wildfly-system-jmx-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/server/integration/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jdr/main/resources/plugins.properties
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jdr/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxrs/main/wildfly-jaxrs-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ee/main/wildfly-ee-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/ws-security/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/main/cxf-core-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/jaxb-jxc-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/domain/configuration/host-master.xml
  inflating: jboss-eap-7.0/domain/configuration/domain.xml
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-LightItalic.ttf
  inflating: jboss-eap-7.0/bin/domain.bat
  inflating: jboss-eap-7.0/bin/jboss-cli.bat
  inflating: jboss-eap-7.0/bin/jdr.bat
  inflating: jboss-eap-7.0/bin/standalone.conf
  inflating: jboss-eap-7.0/bin/standalone.conf.bat
  inflating: jboss-eap-7.0/bin/standalone.sh
  inflating: jboss-eap-7.0/jboss-modules.jar
  inflating: jboss-eap-7.0/docs/schema/jboss-deployment-structure-1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mail_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-facesconfig_1_2.xsd
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/js/main/undertow-js-1.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/mod_cluster/undertow/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/infinispan/spi/main/wildfly-clustering-infinispan-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/main/hibernate-envers-5.0.9.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/web/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/appclient/main/jboss-metadata-appclient-10.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/appclient/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ejb/main/jboss-metadata-ejb-10.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/platform-mbean/main/wildfly-platform-mbean-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/console/eap/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/vault-tool/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/protocol/main/wildfly-protocol-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-BoldItalic.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/wsprovide/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/common/main/jbossws-common-tools-1.2.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-undertow-httpspi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting/main/jboss-remoting-4.0.18.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-bindings-object-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-transports-local-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-transports-http-hc-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-xjc-boolean-3.0.5.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-ws-rm-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/james/mime4j/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-commons-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/ra/main/artemis-service-extensions-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/wss4j-ws-security-policy-stax-2.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jberet/jberet-core/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jaxen/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javaee/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/nu/xom/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/main/service-loader-resources/META-INF/services/javax.script.ScriptEngineFactory
  inflating: jboss-eap-7.0/modules/system/layers/base/sun/scripting/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/main/service-loader-resources/META-INF/services/java.sql.Driver
  inflating: jboss-eap-7.0/modules/system/layers/base/sun/jdk/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/gnu/getopt/main/java-getopt-1.0.13.redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/asm/asm/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/json/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/sql/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/interceptor/api/main/jboss-interceptors-api_1.2_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/mail/api/main/javax.mail-1.5.5.redhat-1.jar
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-ExtraBoldItalic.ttf
  inflating: jboss-eap-7.0/docs/schema/batch-jberet_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-iiop-openjdk_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-undertow_3_1.xsd
  inflating: jboss-eap-7.0/docs/schema/ejb-jar_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-deployment-scanner_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_8.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_7.xsd
  inflating: jboss-eap-7.0/docs/schema/application-client_7.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_1_4.xsd
  inflating: jboss-eap-7.0/docs/licenses/eclipse public license, version 1.0 - epl-1.0.txt
  inflating: jboss-eap-7.0/docs/licenses/jdom license - jdom-1.0.txt
  inflating: jboss-eap-7.0/docs/licenses/cddl 1.1 - cddl+gpl_1_1.html
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/core/main/undertow-core-1.3.21.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/connector/main/wildfly-connector-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/httpmime-4.5.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/httpcore-nio-4.4.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jstl/api/main/jboss-jstl-api_1.2_spec-1.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/istack/main/istack-commons-tools-2.21.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/istack/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xsom/main/xsom-20140925.0.0.redhat-1.jar
  inflating: jboss-eap-7.0/bin/jconsole.bat
  inflating: jboss-eap-7.0/bin/appclient.conf.bat
  inflating: jboss-eap-7.0/bin/add-user.bat
  inflating: jboss-eap-7.0/bin/common.ps1
  inflating: jboss-eap-7.0/bin/domain.conf.ps1
  inflating: jboss-eap-7.0/bin/domain.conf.bat
  inflating: jboss-eap-7.0/bin/jdr.sh
  inflating: jboss-eap-7.0/bin/appclient.sh
  inflating: jboss-eap-7.0/bin/standalone.ps1
  inflating: jboss-eap-7.0/LICENSE.txt
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-azure-ha.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-as-remoting_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mod-cluster_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/j2ee_web_services_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-threads_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/j2ee_jaxrpc_mapping_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-batch-jberet_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cmp_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jacorb_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-naming_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-datasources_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-pojo_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jgroups_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jsr77_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_7.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-container-interceptors_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ee_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-security_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-webservices_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-security_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jgroups_4_0.xsd
  inflating: jboss-eap-7.0/docs/schema/trans-timeout-1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-picketlink-federation_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_web_services_client_1_2.xsd
  inflating: jboss-eap-7.0/docs/licenses/gnu general public license, version 2 with the classpath exception - gpl-2.0-ce.txt
  inflating: jboss-eap-7.0/docs/licenses/licenses.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/websocket/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/json/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/singleton/main/wildfly-clustering-singleton-api-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/infinispan/main/wildfly-clustering-web-infinispan-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/java-support-7.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/bindings/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/main/picketlink-impl-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/envers/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/engine/main/hibernate-search-engine-5.5.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/process-controller/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security-api/main/wildfly-core-security-api-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security-api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security-api/main/wildfly-security-api-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security-api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cmp/main/wildfly-cmp-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cli/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/remoting/main/wildfly-remoting-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-xjc-dv-3.0.5.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-ws-addr-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-services-ws-discovery-api-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/velocity/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xerces/main/xercesImpl-2.11.0.SP4-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/jsp/main/jastow-2.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketbox/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/antlr/main/antlr-2.7.7.redhat-7.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/main/xnio-api-3.3.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/weld/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-management/main/wildfly-domain-management-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xerces/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/neethi/main/neethi-3.0.3.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/beanutils/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/cli/main/commons-cli-1.2.0.redhat-10.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/lucene-queryparser-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/xmlschema/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/jdt/ecj/main/ecj-4.4.2.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/servlet/main/undertow-servlet-1.3.21.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/azure/main/jgroups-azure-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/azure/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/ext/main/slf4j-ext-1.7.7.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/schema/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/api/main/picketlink-idm-api-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/client/main/hornetq-jms-client-2.4.7.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/main/jboss-negotiation-spnego-3.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jandex/main/jandex-2.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/api/main/ironjacamar-core-api-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/main/ironjacamar-validator-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/genericjms/main/generic-jms-ra-jar-1.0.7.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logmanager/main/jboss-logmanager-2.0.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/sar/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/management-client-content/main/wildfly-management-client-content-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/index.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/index_win.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-Bold.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-wsdl-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-tools-java2ws-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/httpasyncclient-4.1.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-hqclient-protocol-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-cli-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jaxen/main/jaxen-1.1.6.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/annotation/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/activation/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/el/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/ejb/api/main/jboss-ejb-api_3.2_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/classmate/main/classmate-1.1.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/jsf-impl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/fastinfoset/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/txw2/main/txw2-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/jaxb-core-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/standalone/configuration/logging.properties
  inflating: jboss-eap-7.0/standalone/configuration/mgmt-users.properties
  inflating: jboss-eap-7.0/standalone/configuration/application-users.properties
  inflating: jboss-eap-7.0/version.txt
  inflating: jboss-eap-7.0/domain/configuration/host-slave.xml
  inflating: jboss-eap-7.0/domain/configuration/mgmt-groups.properties
  inflating: jboss-eap-7.0/domain/configuration/application-roles.properties
  inflating: jboss-eap-7.0/domain/configuration/logging.properties
  inflating: jboss-eap-7.0/domain/configuration/application-users.properties
  inflating: jboss-eap-7.0/welcome-content/images/eap_bg.png
  inflating: jboss-eap-7.0/welcome-content/images/product_title.png
  inflating: jboss-eap-7.0/welcome-content/index.html
  inflating: jboss-eap-7.0/welcome-content/index_noconsole.html
  inflating: jboss-eap-7.0/welcome-content/favicon.ico
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-Bold.ttf
  inflating: jboss-eap-7.0/bin/appclient.conf
  inflating: jboss-eap-7.0/bin/standalone.conf.ps1
  inflating: jboss-eap-7.0/bin/wsprovide.bat
  inflating: jboss-eap-7.0/bin/appclient.conf.ps1
  inflating: jboss-eap-7.0/bin/wsconsume.sh
  inflating: jboss-eap-7.0/bin/jboss-cli.xml
  inflating: jboss-eap-7.0/appclient/configuration/appclient.xml
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-minimalistic.xml
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-gossip-full-ha.xml
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-azure-full-ha.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jmx_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-security_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging-deployment_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-web_7_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-naming_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/application_7.xsd
  inflating: jboss-eap-7.0/docs/schema/web-app_3_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jdr_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/shared-session-config_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-picketlink-idm_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-config_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_4_0.xsd
  inflating: jboss-eap-7.0/docs/licenses/apache software license, version 2.0 - apache-2.0.txt
  inflating: jboss-eap-7.0/modules/system/layers/base/net/jcip/main/jcip-annotations-1.0.0.redhat-8.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/el/main/javax.el-impl-3.0.1.b08-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-xmlsec-impl-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/cdi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/main/hibernate-core-5.0.9.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/js/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/enterprise/concurrent/main/javax.enterprise.concurrent-1.0.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-core-asl/main/jackson-core-asl-1.9.13.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/jdbcadapters/main/ironjacamar-jdbc-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller-client/main/wildfly-controller-client-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-Italic.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-ws-policy-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/james/mime4j/main/apache-mime4j-0.6.0.redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/codemodel-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/bin/jboss-cli.ps1
  inflating: jboss-eap-7.0/bin/vault.ps1
  inflating: jboss-eap-7.0/bin/wsprovide.ps1
  inflating: jboss-eap-7.0/bin/standalone.bat
  inflating: jboss-eap-7.0/bin/init.d/jboss-eap.conf
  inflating: jboss-eap-7.0/bin/domain.ps1
  inflating: jboss-eap-7.0/bin/service.bat
  inflating: jboss-eap-7.0/bin/client/README-CLI-JCONSOLE.txt
  inflating: jboss-eap-7.0/bin/client/README-EJB-JMS.txt
  inflating: jboss-eap-7.0/bin/client/jboss-client.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/core/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/enterprise/concurrent/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-jaxrs/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-mapper-asl/main/jackson-mapper-asl-1.9.13.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/host-controller/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/weld/main/wildfly-weld-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-transports-jms-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/io/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/collections/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/logging/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/beanutils/main/commons-beanutils-1.9.2.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-core-client-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/jaxb-runtime-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/ch/qos/cal10n/main/cal10n-api-0.8.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/elytron/main/wildfly-elytron-1.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/main/wildfly-batch-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/scannotation/scannotation/main/scannotation-1.0.3.redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-saml-impl-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/sasl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logmanager/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/nio/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/nio/main/xnio-nio-3.3.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/iiop-client/main/jboss-iiop-client-1.0.0.Final-redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/main/jboss-logging-3.3.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/network/main/wildfly-network-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/pojo/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/naming/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/naming/main/wildfly-naming-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/protocol/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/host-controller/main/wildfly-host-controller-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-jms-client-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/nu/xom/main/xom-1.2.10.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/websocket/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/github/spullara/mustache/main/compiler-0.8.13.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/messaging/saaj/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/bind/main/jaxb-xjc-2.2.11.redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/undertow/jsp/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/glassfish/javax/el/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/jboss/main/wildfly-clustering-marshalling-jboss-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/common/main/wildfly-common-1.1.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/io/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/clustering/singleton/main/wildfly-clustering-singleton-extension-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/jberet/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/picketlink/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/undertow/main/wildfly-undertow-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ear/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/common/main/jboss-metadata-common-10.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-scanner/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/mail/main/wildfly-mail-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/hibernate/4/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/openjpa/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/jgroups/main/wildfly-clustering-jgroups-extension-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-management/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/server/main/wildfly-server-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jsoup/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/javassist/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jberet/jberet-core/main/jberet-core-1.2.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/github/relaxng/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/classmate/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/jackson-jaxrs-base-2.5.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/google/guava/main/guava-18.0.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/ch/qos/cal10n/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/mod_cluster/undertow/main/wildfly-mod_cluster-undertow-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/api/main/wildfly-clustering-api-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/infinispan/main/wildfly-clustering-ee-infinispan-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/server/main/wildfly-clustering-server-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-profile-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-xmlsec-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/cdi/main/hibernate-validator-cdi-5.2.4.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/engine/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/serialization-avro/main/hibernate-search-serialization-avro-5.5.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate5/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/4.1/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/5.0/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jdom/main/jdom-1.1.3.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/invocation/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/main/jboss-negotiation-common-3.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/xacml/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jandex/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remote-naming/main/jboss-remote-naming-2.0.4.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/common-beans/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting-jmx/main/remoting-jmx-2.0.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jsapi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-validator-provider-11/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/iiop-client/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/jul-to-slf4j-stub/main/jul-to-slf4j-stub-1.0.1.Final-redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/jul-to-slf4j-stub/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/logging/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/compensations/main/compensations-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/core/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/threads/main/jboss-threads-2.2.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxr/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/main/wildfly-jpa-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/images/header_bg.png
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/noConsoleForSlaveDcError.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/favicon.ico
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-SemiboldItalic.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-udp/main/jbossws-cxf-transports-udp-5.1.3.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-client/main/jbossws-cxf-client-5.1.3.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/services-sts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-tools-wsdlto-core-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-xjc-ts-3.0.5.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-management-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-xjc-bug986-3.0.5.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-frontend-jaxws-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/lucene-queries-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/javassist/main/javassist-3.18.1.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/netty/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/security/manager/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/infinispan/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/service/main/wildfly-clustering-service-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/spi/main/wildfly-clustering-ee-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ee/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/server/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/spi/main/wildfly-clustering-ejb-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/iiop-openjdk/main/wildfly-iiop-openjdk-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-atom-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jettison-provider/main/resteasy-jettison-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/main/restat-api-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/integration/ext-content/main/bundled/jboss-seam-int.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jacorb/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf/main/wildfly-jsf-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/pojo/main/wildfly-pojo-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-Light.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-tools-common-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-journal-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/wss4j-ws-security-common-2.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-SemiboldItalic.ttf
  inflating: jboss-eap-7.0/docs/schema/jboss-weld-1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-picketlink-federation_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-app_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jsf_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/application_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mod-cluster_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jpa_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_5.xsd
  inflating: jboss-eap-7.0/modules/system/layers/base/org/bouncycastle/main/bcmail-jdk15on-1.52.0.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/mod_cluster/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/security/manager/main/wildfly-security-manager-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/clustering/singleton/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/rts/main/wildfly-rts-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/jberet/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/yaml/snakeyaml/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/scannotation/scannotation/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-xacml-impl-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/impl/main/jbosgi-xservice.properties
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/impl/main/slf4j-jboss-logmanager-1.0.3.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/common/main/picketlink-common-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/commons-annotations/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/jipijapa-hibernate5/main/jipijapa-hibernate5-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/joda/time/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/dom4j/main/dom4j-1.6.1.redhat-7.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/main/ironjacamar-deployers-common-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/main/ironjacamar-core-impl-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/eap/dir.index
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/eap/dir/META-INF/MANIFEST.MF
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/product/eap/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/cli/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/main/jbossws-cxf-resources-5.1.3.SP1-redhat-1-wildfly1000.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jdr/main/wildfly-jdr-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/appclient/main/wildfly-appclient-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/threads/main/wildfly-threads-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/main/wildfly-domain-http-error-context-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cmp/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/cli/main/wildfly-cli-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/istack/main/istack-commons-runtime-2.21.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xsom/main/module.xml
  inflating: jboss-eap-7.0/standalone/configuration/standalone.xml
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-Italic.ttf
  inflating: jboss-eap-7.0/docs/schema/ejb-jar_3_1.xsd
  inflating: jboss-eap-7.0/modules/system/layers/base/org/bouncycastle/main/bcpkix-jdk15on-1.52.0.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/integration/main/narayana-jts-integration-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb-client/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ear/main/jboss-metadata-ear-10.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/metadata/ejb/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/common-beans/main/jboss-common-beans-2.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/impl/main/ironjacamar-common-impl-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jettison-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jsapi/main/resteasy-jsapi-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/main/restat-integration-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/core/main/weld-core-impl-2.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-selector-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-dto-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/wss4j-ws-security-dom-2.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/stream/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/bind/api/main/jboss-jaxb-api_2.2_spec-1.0.4.Final-redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/github/spullara/mustache/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/github/relaxng/main/relaxngDatatype-2011.1.0.redhat-10.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-core/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-databind/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-annotations/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-annotations/main/jackson-annotations-2.5.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/h2database/h2/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/jsf-impl/main/jsf-impl-2.2.12.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/io/netty/main/netty-all-4.0.32.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ejb3/main/timers/timer-sql.properties
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ejb3/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/process-controller/main/wildfly-process-controller-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xts/main/jbosstxbridge-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jaxbintros/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb/remote/protocol/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb3/main/jboss-ejb3-ext-api-2.2.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb3/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/saaj-impl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-server/main/jbossws-cxf-server-5.1.3.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xml-resolver/main/xml-resolver-1.2.0.redhat-11.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/ws-security/main/cxf-rt-ws-security-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/neethi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/santuario/xmlsec/main/xmlsec-2.0.5.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/batch/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/jms/api/main/jboss-jms-api_2.0_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/rpc/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/soap/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/soap/api/main/jboss-saaj-api_1.3_spec-1.0.3.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/ws/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/management/j2ee/api/main/jboss-j2eemgmt-api_1.1_spec-1.0.1.Final-redhat-4.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/management/j2ee/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/validation/api/main/validation-api-1.1.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/activation/api/main/activation-1.1.1.redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/persistence/api/main/jbosgi-xservice.properties
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/persistence/api/main/hibernate-jpa-2.1-api-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/fastinfoset/main/FastInfoset-1.2.13.redhat-1.jar
  inflating: jboss-eap-7.0/standalone/configuration/mgmt-groups.properties
  inflating: jboss-eap-7.0/standalone/configuration/standalone-full.xml
  inflating: jboss-eap-7.0/standalone/configuration/application-roles.properties
  inflating: jboss-eap-7.0/standalone/configuration/standalone-full-ha.xml
  inflating: jboss-eap-7.0/standalone/configuration/standalone-ha.xml
  inflating: jboss-eap-7.0/standalone/deployments/README.txt
  inflating: jboss-eap-7.0/domain/configuration/host.xml
  inflating: jboss-eap-7.0/domain/configuration/mgmt-users.properties
  inflating: jboss-eap-7.0/domain/configuration/default-server-logging.properties
  inflating: jboss-eap-7.0/welcome-content/images/prod_logo.png
  inflating: jboss-eap-7.0/welcome-content/images/prod_name.png
  inflating: jboss-eap-7.0/welcome-content/images/header_bg.png
  inflating: jboss-eap-7.0/welcome-content/eap.css
  inflating: jboss-eap-7.0/welcome-content/noconsole.html
  inflating: jboss-eap-7.0/welcome-content/noredirect.html
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-Light.ttf
  inflating: jboss-eap-7.0/bin/appclient.bat
  inflating: jboss-eap-7.0/bin/wsconsume.ps1
  inflating: jboss-eap-7.0/bin/init.d/jboss-eap-rhel.sh
  inflating: jboss-eap-7.0/bin/.jbossclirc
  inflating: jboss-eap-7.0/bin/jboss-cli-logging.properties
  inflating: jboss-eap-7.0/bin/add-user.properties
  inflating: jboss-eap-7.0/bin/client/jboss-cli-client.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/fusesource/jansi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/api/main/wildfly-clustering-marshalling-api-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/spi/main/wildfly-clustering-jgroups-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/undertow/main/wildfly-clustering-web-undertow-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/security/manager/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/io/main/wildfly-io-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/picketlink/main/wildfly-picketlink-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting-jmx/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/jdbcadapters/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/api/main/ironjacamar-common-spi-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ironjacamar/api/main/ironjacamar-common-api-1.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/version/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jmx/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ee/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/self-contained/main/wildfly-self-contained-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/self-contained/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/appclient/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/console/eap/release-stream-2.8.24.Final-redhat-1-resources.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/bouncycastle/main/bcprov-jdk15on-1.52.0.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-ws-mex-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-tools-wsdlto-frontend-jaxws-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/collections/main/commons-collections-3.2.2.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/wss4j-ws-security-stax-2.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/orb/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/transaction/api/main/jboss-transaction-api_1.2_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/transaction/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/batch/api/main/jboss-batch-api_1.0_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/faces/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/jws/api/main/jsr181-api-1.0.0.MR1-redhat-8.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/jws/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/rpc/api/main/jboss-jaxrpc-api_1.1_spec-1.0.1.Final-redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/bind/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/xml/ws/api/main/jboss-jaxws-api_2.2_spec-2.0.2.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/validation/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/concurrent/api/main/jboss-concurrency-api_1.0_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/el/api/main/jboss-el-api_3.0_spec-1.0.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/jackson-jaxrs-json-provider-2.5.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-databind/main/jackson-databind-2.5.4.redhat-1.jar
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-xts.xml
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-jts.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-as-security_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-io_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-deployment-structure-1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_web_services_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jaxrs_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/j2ee_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ee_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_web_services_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-undertow_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-singleton_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-threads_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_web_services_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-deployment-scanner_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-fragment_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/ejb-jar_3_2.xsd
  inflating: jboss-eap-7.0/docs/schema/web-common_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-deployment-scanner_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-client_6_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_2_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/singleton-deployment_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jsp_2_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cli_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-naming_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-jca_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ee_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-naming_1_1.xsd
  inflating: jboss-eap-7.0/docs/licenses/the apache software license, version 2.0 - license-2.0.txt
  inflating: jboss-eap-7.0/docs/licenses/apache license, version 2.0 - apache2.0.php
  inflating: jboss-eap-7.0/docs/licenses/h2 license, version 1.0 - h2.txt
  inflating: jboss-eap-7.0/docs/licenses/the gnu lesser general public license, version 2.1 - lgpl-2.1.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/client/hotrod/main/infinispan-client-hotrod-8.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/jcl-over-slf4j/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/config/main/picketlink-config-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/core/api/main/picketlink-api-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/serialization-avro/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jms/main/hibernate-search-backend-jms-5.5.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jgroups/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/commons-annotations/main/hibernate-commons-annotations-5.0.1.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketbox/main/picketbox-infinispan-4.9.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/client/main/hornetq-commons-2.4.7.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/river/main/jboss-marshalling-river-1.4.10.Final-redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remote-naming/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/deployers/jboss-service-deployer/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/aesh/main/aesh-0.66.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxb-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-cdi/main/resteasy-cdi-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-multipart-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-atom-provider/main/resteasy-atom-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/compensations/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/main/restat-util-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/classfilewriter/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/api/main/weld-api-2.3.0.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/msc/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jacorb/main/wildfly-jacorb-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/webservices/main/wildfly-webservices-server-integration-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/xts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/messaging/main/wildfly-messaging-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-bindings-xml-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-tools-validator-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-databinding-jaxb-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/santuario/xmlsec/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/codec/main/commons-codec-1.10.0.redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-server-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/bin/jconsole.ps1
  inflating: jboss-eap-7.0/bin/add-user.sh
  inflating: jboss-eap-7.0/bin/add-user.ps1
  inflating: jboss-eap-7.0/bin/domain.sh
  inflating: jboss-eap-7.0/bin/wsprovide.sh
  inflating: jboss-eap-7.0/bin/appclient.ps1
  inflating: jboss-eap-7.0/bin/domain.conf
  inflating: jboss-eap-7.0/appclient/configuration/logging.properties
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-gossip-ha.xml
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-picketlink.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-client_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jgroups_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/web-jsptaglibrary_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ee_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cli_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jmx_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_2_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-deployment-dependencies-1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-txn_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-remoting_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-weld-1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-ejb3_4_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-webservices_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-facesconfig_2_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mail_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-xts_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/persistence_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/persistence_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_1_5.xsd
  inflating: jboss-eap-7.0/docs/licenses/gpl - gplv2+ce.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/fusesource/jansi/main/jansi-1.11.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/rts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/request-controller/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/bean-validation/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/messaging-activemq/main/artemis-wildfly-integration-1.0.2.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-xacml-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-security-impl-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/woodstox/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jgroups/main/jgroups-3.6.8.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/internal/main/lucene-misc-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/lib/linux-i686/libartemis-native-32.so
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/lib/linux-x86_64/libartemis-native-64.so
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/protocol/hornetq/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/ra/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/wss4j-bindings-2.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/wss4j-policy-2.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/security/jacc/api/main/jboss-jacc-api_1.5_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/resource/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/orb/api/main/openjdk-orb-8.0.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/bouncycastle/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/marshalling/jboss/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/api/main/wildfly-clustering-web-api-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-security-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/main/slf4j-api-1.7.7.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/ext/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/jcl-over-slf4j/main/jcl-over-slf4j-1.7.7.redhat-3.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/config/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/main/picketlink-idm-impl-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/vfs/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ejb-client/main/jboss-ejb-client-2.1.4.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jpa/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/infinispan/main/wildfly-clustering-infinispan-extension-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/internal/main/lucene-backward-codecs-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/api/main/cdi-api-1.2.0.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/core/jackson-core/main/jackson-core-2.5.4.redhat-1.jar
  inflating: jboss-eap-7.0/bin/wsconsume.bat
  inflating: jboss-eap-7.0/bin/vault.sh
  inflating: jboss-eap-7.0/bin/product.conf
  inflating: jboss-eap-7.0/bin/vault.bat
  inflating: jboss-eap-7.0/bin/jconsole.sh
  inflating: jboss-eap-7.0/bin/jdr.ps1
  inflating: jboss-eap-7.0/bin/jboss-cli.sh
  inflating: jboss-eap-7.0/bin/launcher.jar
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-rts.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-resource-adapter-binding_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jca_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/application-client_6.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jpa_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-config_4_1.xsd
  inflating: jboss-eap-7.0/docs/licenses/cddl+gpl license - cddl+gpl_1_1.html
  inflating: jboss-eap-7.0/docs/licenses/gnu lesser general public license - lgpl-3.0.txt
  inflating: jboss-eap-7.0/docs/licenses/the mit license - mit.txt
  inflating: jboss-eap-7.0/docs/licenses/public domain - cc0-1.0.txt
  inflating: jboss-eap-7.0/docs/licenses/mozilla public license, version 1.1 - mpl-1.1.txt
  inflating: jboss-eap-7.0/modules/system/layers/base/ibm/jdk/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/naming/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/jgroups/api/main/wildfly-clustering-jgroups-api-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/spi/main/wildfly-clustering-spi-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-core-asl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-xc/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-mapper-asl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jettison/main/jettison-1.3.3.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/slf4j/impl/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/idm/schema/main/picketlink-idm-simple-schema-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/main/picketlink-federation-2.5.5.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/logging/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jmx/main/wildfly-jmx-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/connector/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/xts/main/wildfly-xts-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/messaging/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/modcluster/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/ejb3/main/wildfly-ejb3-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-BoldItalic.ttf
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-genericjms.xml
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-ec2-full-ha.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jacorb_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-common_5_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-io_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-resource-adapters_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-batch_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-pool_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-ejb-timer_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jaxr_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-weld_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-web_7_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jgroups_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-app_2_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jgroups_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-weld_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jacorb_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/module-1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-weld_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-security_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jbxb_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-picketlink-idm_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-datasources_4_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-security-role_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-web_8_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-delivery-active_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jaxr_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/web-app_2_5.xsd
  inflating: jboss-eap-7.0/docs/licenses/creative commons attribution 2.5 - jcip-cc-by-2.5.txt
  inflating: jboss-eap-7.0/docs/licenses/common public license, version 1.0 - cpl-1.0.txt
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/jdbc/main/infinispan-cachestore-jdbc-8.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/request-controller/main/wildfly-request-controller-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/undertow/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/jberet/main/wildfly-jberet-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-core-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-xacml-saml-impl-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-xacml-saml-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jackson/jackson-jaxrs/main/jackson-jaxrs-1.9.13.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/jettison/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/codehaus/woodstox/main/stax2-api-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketlink/federation/bindings/main/picketlink-wildfly8-2.5.5.SP1-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/picketbox/main/picketbox-4.9.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson2-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/jose-jwt/main/jose-jwt-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-crypto/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-spring/main/bundled/resteasy-spring-jar/resteasy-spring-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-validator-provider-11/main/resteasy-validator-provider-11-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/txframework/main/txframework-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/weld/probe/main/weld-probe-core-2.3.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-ExtraBoldItalic.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/remoting/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xts/main/jbossxts-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/codec/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/pool/main/commons-pool-1.6.0.redhat-9.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/cli/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/openjpa/main/jipijapa-openjpa-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/openjpa/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/lucene-facet-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/internal/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/httpcomponents/main/httpcore-4.4.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/protocol/hornetq/main/artemis-hornetq-protocol-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/ra/main/artemis-ra-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/xmlschema/main/xmlschema-core-2.2.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/interceptor/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jsp/api/main/jboss-jsp-api_2.3_spec-1.0.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/security/auth/message/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/security/jacc/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/wsdl4j/api/main/wsdl4j-1.6.3.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/ws/rs/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/persistence/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/websocket/api/main/jboss-websocket-api_1.1_spec-1.1.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/google/guava/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/microsoft/azure/storage/main/azure-storage-4.0.0.redhat-1.jar
  inflating: jboss-eap-7.0/docs/examples/configs/standalone-ec2-ha.xml
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_6.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cli_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-deployment-structure-1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/j2ee_web_services_client_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_6.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-iiop_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-remoting_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-jsptaglibrary_2_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mod-cluster_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/java-properties_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-cli_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-webservices_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-datasources_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/web-fragment_3_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-naming_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_2_0.xsd
  inflating: jboss-eap-7.0/docs/licenses/the werken company license - license.html
  inflating: jboss-eap-7.0/docs/licenses/the bsd license - bsd.txt
  inflating: jboss-eap-7.0/docs/licenses/gnu lesser general public license, version 2 - lgpl-2.0.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/remote/main/infinispan-cachestore-remote-8.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/bean-validation/main/wildfly-bean-validation-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/yaml/snakeyaml/main/snakeyaml-1.15.0.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/4.3/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/validator/main/hibernate-validator-5.2.4.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/netty/netty-xnio-transport/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jboss-transaction-spi/main/jboss-transaction-spi-7.3.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxrs/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxb-provider/main/resteasy-jaxb-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/rts/main/restat-bridge-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/dmr/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/classfilewriter/main/jboss-classfilewriter-1.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-interface/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller-client/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-repository/main/wildfly-deployment-repository-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/jgroups/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/clustering/common/main/wildfly-clustering-common-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web-common/main/wildfly-web-common-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/logging/main/wildfly-logging-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-client/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/common/main/jbossws-common-3.1.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-security-saml-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-transports-http-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/lucene/main/lucene-analyzers-common-5.3.1.redhat-1.jar
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jmx_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jbossws-jaxws-config_4_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-cache_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-jpa_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/module-1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-bean-validation_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb3-2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jsp_2_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jacorb_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mod-cluster_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-client_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cli_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-picketlink-idm_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-xts_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-messaging-activemq_1_0.xsd
  inflating: jboss-eap-7.0/docs/licenses/common development and distribution license - cddl.txt
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/jdbc/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/web/undertow/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/main/hibernate-java8-5.0.9.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/backend-jms/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/search/orm/main/hibernate-search-orm-5.5.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/stdio/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/log4j/logmanager/main/log4j-jboss-logmanager-1.1.2.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/mail/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/standalone/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/security/main/wildfly-security-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/server/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/threads/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir.index
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/noConsoleForAdminModeError.html
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/images/prod_logo.png
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/images/eap_bg.png
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/images/prod_name.png
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/images/product_title.png
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/eap.css
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-LightItalic.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-client/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-undertow/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-udp/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/sts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-factories/main/jbossws-cxf-factories-5.1.3.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/api/main/jbossws-api-1.0.3.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/staxmapper/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/avro/main/avro-1.7.6.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-tools-wsdlto-databinding-jaxb-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/velocity/main/velocity-1.7.0.redhat-5.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/ws/security/main/jasypt-1.9.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jsp/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/jstl/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/servlet/api/main/jboss-servlet-api_3.1_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/enterprise/concurrent/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/annotation/api/main/jboss-annotations-api_1.2_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/ejb/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/inject/api/main/javax.inject-1.0.0.redhat-6.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/inject/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/ws/rs/api/main/jboss-jaxrs-api_2.0_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/txw2/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/sun/xml/messaging/saaj/main/saaj-impl-1.3.16.SP1-redhat-6.jar
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-Semibold.ttf
  inflating: jboss-eap-7.0/docs/schema/wildfly-datasources_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cmp_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/module-1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-undertow_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jsp_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-remoting_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-undertow_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/module-1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-deployment-structure-1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-pojo_7_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-facelettaglibrary_2_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-mail_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jbossws-web-services_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-ejb3_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-threads_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-infinispan_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-logging_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/module-1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jacorb_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-request-controller_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb3-spec-2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-web_7_2.xsd
  inflating: jboss-eap-7.0/docs/schema/ejb-jar_2_1.xsd
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/commons/main/infinispan-commons-8.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/main/jboss-negotiation-ntlm-3.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/negotiation/main/jboss-negotiation-extras-3.0.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/aesh/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jts/main/narayana-jts-idlj-5.2.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-xjc-runtime-3.0.5.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-databinding-aegis-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/gnu/getopt/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/asm/asm/main/asm-3.3.1.redhat-12.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/mail/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/security/auth/message/api/main/jboss-jaspi-api_1.1_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/wsdl4j/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/rmi/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/resource/api/main/jboss-connector-api_1.7_spec-1.0.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/jms/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/javax/faces/api/main/jboss-jsf-api_2.2_spec-2.2.12.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/main/infinispan-core-8.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/docs/schema/user-roles_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jndi-binding-service_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-cli_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ee_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-mail_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-ejb3_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/web-common_3_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-undertow_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-webservices_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-config_2_2.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-resource-adapters_4_0.xsd
  inflating: jboss-eap-7.0/docs/licenses/apache license 2.0 - license-2.0.txt
  inflating: jboss-eap-7.0/docs/licenses/eclipse public license, version 1.0 - epl-v10.html
  inflating: jboss-eap-7.0/docs/licenses/eclipse distribution license, version 1.0 - edl-1.0.txt
  inflating: jboss-eap-7.0/JBossEULA.txt
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hibernate/main/hibernate-entitymanager-5.0.9.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jaxrs/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf-injection/main/wildfly-jsf-injection-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/transactions/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller/main/wildfly-controller-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/welcome-content/fonts/OpenSans-ExtraBold.ttf
  inflating: jboss-eap-7.0/docs/schema/jboss-as-datasources_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/orm_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-web_2_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jsp_2_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-resource-adapters_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-app_7_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-client_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-jca_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_web_services_client_1_4.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-messaging_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-clustering_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-resource-adapters_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-jca_4_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-common_6_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-ejb-delivery-active_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/javaee_web_services_client_1_3.xsd
  inflating: jboss-eap-7.0/docs/schema/application_6.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-security-manager_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jmx_1_2.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-undertow_3_0.xsd
  inflating: jboss-eap-7.0/docs/schema/web-partialresponse_2_2.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-txn_1_5.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-picketlink-federation_1_1.xsd
  inflating: jboss-eap-7.0/docs/schema/wildfly-messaging-activemq-deployment_1_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-jacorb_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-as-naming_2_0.xsd
  inflating: jboss-eap-7.0/docs/schema/jboss-web_10_0.xsd
  inflating: jboss-eap-7.0/docs/licenses/gnu lesser general public license, version 2.1 - lgpl-2.1.txt
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/client/hotrod/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/clustering/ejb/infinispan/main/wildfly-clustering-ejb-infinispan-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/messaging-activemq/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/batch/jberet/main/wildfly-batch-jberet-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/opensaml/main/opensaml-saml-api-3.1.1.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/hornetq/client/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/joda/time/main/joda-time-2.7.0.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson-provider/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-crypto/main/resteasy-crypto-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/integration/ext-content/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/threads/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/msc/main/jboss-msc-1.2.6.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsf-injection/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web-common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/core-security/main/wildfly-core-security-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/controller/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/web/main/wildfly-web-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-Semibold.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/spi/main/jbossws-spi-3.1.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/services-sts/main/cxf-services-sts-core-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-bindings-coloc-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/cxf/impl/main/cxf-rt-frontend-simple-3.1.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/pool/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/commons/lang/main/commons-lang-2.6.0.redhat-6.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-native-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/activemq/artemis/main/artemis-jms-server-1.1.0.SP16-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/jdt/ecj/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/persistence/main/jipijapa-eclipselink-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/eclipse/persistence/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jsoup/main/jsoup-1.8.3.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/jackson-module-jaxb-annotations-2.5.4.redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/com/microsoft/azure/storage/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/com/h2database/h2/main/h2-1.3.173.redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/infinispan/cachestore/remote/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/iiop-openjdk/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/embedded/main/wildfly-embedded-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/wildfly/extension/messaging-activemq/main/wildfly-messaging-activemq-7.0.0.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/dom4j/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/omg/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/modules/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/stdio/main/jboss-stdio-1.0.2.GA-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/log4j/logmanager/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/container/spi/main/mod_cluster-container-spi-1.3.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/container/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/mod_cluster/core/main/mod_cluster-core-1.3.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/marshalling/river/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/invocation/main/jboss-invocation-1.4.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/security/xacml/main/jbossxacml-2.0.8.Final-redhat-6.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/genericjms/main/META-INF/ra.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/genericjms/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/sasl/main/jboss-sasl-1.0.5.Final-redhat-2.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xnio/netty/netty-xnio-transport/main/netty-xnio-transport-0.1.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jboss-transaction-spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jackson2-provider/main/resteasy-jackson2-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/jose-jwt/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-yaml-provider/main/resteasy-yaml-provider-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/resteasy/resteasy-jaxrs/main/resteasy-client-3.0.16.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/narayana/txframework/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/dmr/main/jboss-dmr-1.3.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/jsr77/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/deployment-scanner/main/wildfly-deployment-scanner-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/system-jmx/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/patching/main/wildfly-patching-2.1.2.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/as/domain-http-error-context/eap/dir/fonts/OpenSans-Regular.ttf
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/xts/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/jaxbintros/main/jboss-jaxb-intros-1.0.2.GA-redhat-7.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-server/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-transports-undertow/main/jbossws-cxf-transports-undertow-5.1.3.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/cxf/jbossws-cxf-factories/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/wsconsume/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/tools/common/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/api/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-client/main/jbossws-cxf-jaspi-5.1.3.SP1-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/spi/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/ws/jaxws-undertow-httpspi/main/jaxws-undertow-httpspi-1.0.1.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/staxmapper/main/staxmapper-1.2.0.Final-redhat-1.jar
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting3/remoting-jmx/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/jboss/remoting3/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/avro/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/log4j/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xml-resolver/main/module.xml
  inflating: jboss-eap-7.0/modules/system/layers/base/org/apache/xalan/main/xalan-2.7.1.redhat-10.jar
  inflating: jboss-eap-7.0/docs/licenses/bouncy castle licence - licence.html
  inflating: jboss-eap-7.0/docs/licenses/the mit license - license.txt
  inflating: jboss-eap-7.0/docs/licenses/lgpl - lgpl.txt

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
