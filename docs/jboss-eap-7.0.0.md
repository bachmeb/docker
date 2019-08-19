# docker - jboss-eap-7.0.0
This guide uses an EC2 VM in AWS

## References
* https://docs.docker.com/get-started/

### Prepare

##### Create a new vm
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
	cat /proc/version
```c
// Linux version 4.1.17-22.30.amzn1.x86_64 (mockbuild@gobi-build-60009) 
// (gcc version 4.8.3 20140911 (Red Hat 4.8.3-9) (GCC) ) 
// #1 SMP Fri Feb 5 23:44:22 UTC 2016
```
##### Check the time
	date
	
##### List the available time zones
	ls /usr/share/zoneinfo/

##### Update the /etc/sysconfig/clock file with your time zone
	sudo nano /etc/sysconfig/clock
```
ZONE="America/New_York"
UTC=false
```
##### Create a symbolic link between /etc/localtime and your time zone file
	sudo ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime

##### Check the time
	date

*[When you launch an instance, it is assigned a hostname that is a form of the private, internal IP address. A typical Amazon EC2 private DNS name looks something like this: ip-12-34-56-78.us-west-2.compute.internal, where the name consists of the internal domain, the service (in this case, compute), the region, and a form of the private IP address. Part of this hostname is displayed at the shell prompt when you log into your instance (for example, ip-12-34-56-78). Each time you stop and restart your Amazon EC2 instance (unless you are using an Elastic IP address), the public IP address changes, and so does your public DNS name, system hostname, and shell prompt. Instances launched into EC2-Classic also receive a new private IP address, private DNS hostname, and system hostname when they're stopped and restarted; instances launched into a VPC don't.](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-hostname.html)*

*[If you have a public DNS name registered for the IP address of your instance (such as webserver.mydomain.com), you can set the system hostname so your instance identifies itself as a part of that domain. This also changes the shell prompt so that it displays the first portion of this name instead of the hostname supplied by AWS (for example, ip-12-34-56-78).](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-hostname.html)*

##### Edit the sysconfig network file
	sudo nano /etc/sysconfig/network

*Rather than set the hostname of this machine to match a fully qualified domain name, this tutorial assumes we want the server to have a unique hostname that is different than any of the hostnames we define in the public DNS file.*

##### Set the HOSTNAME to be any name you would like for the virtual server
	HOSTNAME=[somehost].localdomain

##### Edit the hosts file
	sudo nano /etc/hosts
```
127.0.0.1  [somehost] [somehost].localdomain localhost localhost.localdomain
```

##### Reboot the system to pick up the new hostname and time zone information in all services and applications
	sudo reboot

##### Connect via SSH
```
ssh -i pemfile.pem ec2-user@[ec2.ipa.ddr.ess]
```

##### Check the time
	date

##### Switch to super user
	sudo su
	
##### Check the default options configured for adding a user on this system
	useradd -D
    
##### Create a non-admin user account for yourself
	useradd [your user account name]
    
##### Set your password
	passwd [your user account name]

##### Edit sudoers file
	vim /etc/sudoers
	:$

##### Add your account to sudoers file
	## LET ME SUDO
	[your user account name] ALL=(ALL) ALL

##### Switch to your user account. Be sure to use the -l switch.
	su -l [your user account name]

##### Ask who am I?
	whoami

##### Echo the value of the $HOME environment variable
	echo $HOME
	
##### Go home
	cd ~
	pwd

##### Change the command prompt to show your username at the fqdn
	export PS1='[\u@\H \W]\$'

##### Read your .bashrc file
	cat .bashrc
	
##### Add the PS1 command to your .bashrc file
	echo 'PS1="[\u@\H \W]\$ "'>>~/.bashrc

##### Read your .bashrc file
	cat .bashrc
	
##### Check the hostname of the ec2 instance
	hostname

##### Check the DNS domain name of the ec2 instance
	dnsdomainname

##### Get the IP address of the internal name server
	cat /etc/resolv.conf

##### Check the routing table
	route

##### Check the internal IP address
	ifconfig

##### Check the external IP address
	wget http://ipinfo.io/ip -qO -

##### Run a speed test (just for fun)
```
wget https://raw.github.com/sivel/speedtest-cli/master/speedtest_cli.py
chmod +x speedtest_cli.py
./speedtest_cli.py
```

##### Install system updates
	sudo yum update	


```

sudo yum search docker

