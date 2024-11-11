---
tags:
  - excalidraw
  - RHCSA
  - Redhat
---
We are given 2 nodes (servera, serverb)
- N1 has 15q (serverb)
- N2 has 6q (servera)



![[Drawing 2024-11-11 10.34.27.excalidraw]]

> Root user password for serverb.lab.example.com is redhat

# Assign Hostname and Ip address for your virtual machine.

         Hostname serverb.lab.example.com
         IP Address 172.25.250.11
         Netmask  255.255.255.0
         Gateway 172.25.250.254
         Nameserver 172.25.250.254

> "working in the console -> `serverb`"



#### Commands :
1) Configure the hostname and connection
   ```sh
	 
	 hostname
	 
	 hostnamectl set-hostname serverb.lab.example.com
	 
	 nmcli con show
	 
	 nmcli con modify "Wired connection 1" ipv4.addresses 172.25.250.11/24 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.250.254 ipv4.method manual
	 
	 nmcli con up "Wired connection 1"

	 ping 172.25.250.11

	 ping 172.25.250.254
```

2) Edit ssh Config  
   ```sh
	vim /etc/ssh/sshd_config
```
	**look for PermitRootLogin -> say yes**
```vim
	PermitRootLogin yes
```
3) Reboot the server
   ```sh
	reboot
```

![[Pasted image 20241111110750.png]]


![[Recording 20241111105521.webm]]

# Open Workstation

1) 
   > now ssh into `root@172.25.250.11` from workstation to root user in `serverb` 
```sh
	ssh root@172.25.250.11
```

> **DONT LEAVE SSH **
> *rip @samarth 🥲🥲*


![[Recording 20241111110105.webm]]

2) Create a repository file
   `http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream` 
   `http://classroom.example.com/content/rhel9.0/x86_64/dvd/BaseOS`

#### Commands
1) configuring upstream for dnf/yum installation
   ```sh
	vim /etc/yum.repos.d/local.repo
```
```vim

> 	[repo1]
> 	name=AppStream
> 	baseurl=http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream
> 	gpgcheck=0
> 	enabled=1
> 	
> 	[repo2]
> 	name=BaseOS
> 	baseurl=http://classroom.example.com/content/rhel9.0/x86_64/dvd/BaseOS
> 	gpgcheck=0
> 	enabled=1
```
```sh
	dnf clean all
	dnf repolist all
	dnf install vim
```


![[Recording 20241111113908.webm]]

# Selinux / Web server config

#### Commands
1)  configure http port to use port 82
```sh 
	semanage port -l | grep http
	semanage port -a -t http_port_t -p tcp 82
	semanage port -l | grep http
```
2) allow-list port 82 for tcp connections
   ```sh
	firewall-cmd --permananent --add-port=82/tcp
	firewall-cmd --reload
```
3) install apache-http-server
   ```sh
   dnf install httpd

   vim /etc/httpd/conf/httpd.conf
```
4) change the listen port to `82`  -> `Listen 82`
5) Start web server and curl
   ```sh
   systemctl start httpd
   systemctl enable httpd
   
   curl 172.25.250.11:82
```

![[Recording 20241111115202.webm]]

#  Creating User accounts

#### Configuration
1) A group named *admin*.
2) A user harry who belongs to *admin* as a secondary group.
3) A user natasha who belongs to *admin* as a secondary group.
4) A user sarah who *does not have access to an interactive shell* on the system and who is *not member of admin*.
5) set a password as `redhat` for three users harry, Natasha, sarah.
#### Commands
1) Create users acc to Configuration above
   ```sh
	groupadd admin
	
	useradd -G admin
	useradd -G admin harry
	useradd -G admin natasha
	
	useradd -s /sbin/nologin sarah
	
	passwd --stdin harry 
	> redhat
	passwd --stdin natasha 
	> redhat
	passwd --stdin sarah 
	> redhat
```

![[Recording 20241111123804.webm]]

# Create Collaborative Directory

#### Configuration
1) Group ownership of /common/admin is admin.

2) The directory should be readable, writable and accessible to members of admin, but not to any other user. 
   (It is understood that root has access to all files and directories on the system.)

3) Files created in /common/admin automatically have group ownership set to the admin group.

#### Commands

Adding shared directory and adding permissions for a common directory within the admin grp
   ```sh
	mkdir -p /common/admin/
	
	chgrp admin /common/admin/
	
	chmod 770 /common/admin/
	chmod g+s /common/admin/
	
	ls -ld /common/admin/

	su - harry
	touch /common/admin/file.txt
	ls -la /common/admin/file.txt
```

# AutoFS for Automounting shared Directory


![[Recording 20241111141030.webm]]

#### Commands
```sh
	dnf install autofs
	systemctl start autofs
	systemctl enable autofs

	getent passwd production5 
	su - production5

	vim /etc/auto.master
```
> Edit the file to add the homedir name : `/localhome`     `/etc/auto.misc`

```sh
	vim /etc/auto.misc
```
> Edit the file and add: `production5    -rw,soft,intr      servera.lab.example.com:/user-homes/production5`

```sh
	systemctl restart autofs
```

# Cron Jobs

#### Configuration
1. The user harry must configure cron job that runs daily at 12:30 local time and execute /bin/echo "hello".

2. The user harry must configure cron job that runs every 2 minutes and execute /bin/echo "hello".

3. The user harry must configure cron job that runs daily at 14:23 local time and execute logger message "RHCSA EXAM TRAINING".

4. The user harry must configure cron job that runs daily at 12 noon local time and execute logger message "Cheen dabak dum dim".
5. 
#### Commands
1) open and edit crontab as user harry
   ```sh
	crontab -eu harry
```
2) Add the conditional cron jobs
   ```sh
   30 12 * * * /bin/echo "hello"
   */2 * * * * /bin/echo "hello"
   23 14 * * * logger user.debug "RHCSA EXAM TRAINING"
   00 12 * * * logger user.debug "Cheen dabak dum dim"
```
3) List user jobs
   ```sh
   crontab -lu harry
```