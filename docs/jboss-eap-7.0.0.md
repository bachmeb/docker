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
