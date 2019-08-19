# docker - jboss-eap-7.0.0
This guide uses an EC2 VM in AWS

## References
* https://docs.docker.com/get-started/

### Prepare
#### Download JBoss EAP 7.0.0
* https://developers.redhat.com/products/eap/download

#### Choose the zip file
* https://developers.redhat.com/download-manager/file/jboss-eap-7.0.0.zip

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
sudo nano /etc/sysconfig/clock
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
useradd [your user account name]
```

##### Set your password
```
passwd [your user account name]
```

##### Add your account to the wheel group
```
sudo usermod -aG wheel [your user account name]
```

##### Switch to your user account. Be sure to use the -l switch.
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
wget https://raw.github.com/sivel/speedtest-cli/master/speedtest_cli.py
chmod +x speedtest_cli.py
./speedtest_cli.py
```

#### Install Docker
```
sudo yum search docker
sudo yum install docker
```
