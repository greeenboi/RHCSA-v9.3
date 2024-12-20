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

![[Pasted image 20241111144501.png]]

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

	getent passwd netuser10 

	vim /etc/auto.master
```
> Edit the file to add the homedir name at the **last line** : `/localhome`     `/etc/auto.misc`

```sh
	vim /etc/auto.misc
```
> Edit the file and add: `netuser10    -rw,soft,intr      servera.lab.example.com:/user-homes/production5`

| Permission     | What to add |
| -------------- | ----------- |
| write          | rw          |
| read and write | rw          |
| read alone     | r           |
| read only      | ro          |


```sh
	systemctl restart autofs
```

![[Pasted image 20241111144522.png]]

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

![[Recording 20241111143027.webm]]

# Configure NTP

> Configure your system so that it is an NTP client of classroom.example.com

```sh
	vim /etc/chrony.conf
```
> remove the ntp server line (comment it) and then add
> `server    classroom.example.com    iburst`

```sh
	systemctl restart chronyd
	chronyc sources
```


![[Recording 20241111143339.webm]]

# Locate the Files

> Locate all the files owned by `sarah` and make a copy of them in the given path `/root/find.user`

#### Commands

```sh
	mkdir /root/find.user
	
	find / -user sarah -type f -exec cp {} /root/find.user/ \;
	# The  \ and ; have no space

	ls -la find.user/
```

![[Recording 20241111143833.webm]]

# Find a String

> Find a string `home` in `/etc/passwd` and searching string as been stored in `/root/search.txt`

#### Commands
```
	grep "home" /etc/passwd > /root/search.txt
	nano search.txt
```

![[Recording 20241111144331.webm]]

# Create a User Account 

> Create an user account with `Userid 1326` and user name as `alies`.

#### Commands
```sh
	useradd -u 1326 alies
	echo "meow ;)"
```

![[Recording 20241111144737.webm]]

# Create a tar archive

> 1) Backup the `/var/tmp` as `/root/test.tar.gz` using gzip
> 2) Backup the `/var/tmp` as `/root/test.tar.bz` using bzip
> - zcvf
> - jvcf

#### Commands
1)  install dependencies
   ```sh
	dnf install gzip bzip2
```
2)  package with `tar` cmd using -zcvf
   ```sh
	tar -zcvf /root/text.tar.gz /var/tmp
	ls
```
3)  package with `tar` cmd using -jcvf
   ```sh
	tar -jcvf /root/text.tar.bz2 /var/tmp
	ls
```


![[Recording 20241111145305.webm]]

# Configure the Rootless container as a system start-up service and mount volumes persistently

#### Configuration

![[WhatsApp Image 2024-11-12 at 12.08.54_a36b3c29.jpg]]

a) Create the container as **student** user

b) Run the container by using image <mark style="background: #FFF3A3A6;">admin034/monitor:latest</mark>

c) Create the container as a system start-up service, While reboot it will automatically start the service without any human intervention.

d) The system service should be <mark style="background: #FFF3A3A6;">container-ascii2pdf</mark>.

e) The local directory <mark style="background: #FFF3A3A6;">/opt/files</mark> should be persistently mount on container’s <mark style="background: #FFF3A3A6;">/opt/incoming</mark> directory.

f) The local directory <mark style="background: #FFF3A3A6;">/opt/processed</mark> should be persistently mount on container’s <mark style="background: #FFF3A3A6;">/opt/outgoing</mark> directory.

>Container registry server is `docker.io`
>Use `admin034` as username and `redhat321` as password for container registry

    Note: In working of service starts, any file create/store under the /opt/files automatically creates into pdf on /opt/outgoing directory.
> ssh into root and student and tab switch
#### Commands
1) in root
```sh
	mkdir -p /opt/files /opt/processed
	chown student:student /opt/files /opt/processed

	chmod 777 /opt/files/ /opt/processed
	loginctl enable-linger student	
```
	now ssh into student
1) in student@serverb
   ```sh
    sudo dnf -y install podman
    sudo dnf -y install container-tools
	podman login #mentioned registry
	podman pull registry/admin034/monitor:latest

	podman run -d --name ascii2pdf -v /opt/files:/opt/incoming/:Z -v /opt/processed:/opt/outgoing/:Z monitor:latest
```

3) in student@serverb
```sh
	mkdir -p .config/systemd/user
	cd .config/systemd/user
	podman generate systemd --name ascii2pdf --new --files
	ls
   
	systemctl --user daemon-reload
	systemctl --user start container-ascii2pdf.service
	systemctl --user enable container-ascii2pdf.service
	
```
4) now to test
```sh
	vim /opt/files/data
	files /opt/files/data
	
	files /opt/processed/data
```

![[Recording 20241112120407.webm]]

# Set the Permission

> 1) All new creating files for user `natasha` as `-r--------` as default permission.
> 2) All new creating directories for user `natasha` as `dr-x------` as default permission.

##### Calculating Umask Value
> format : { d rwx rwx rwx  }

(q) `w--rwxrwx`

```txt
w -> 2
rwx -> 7

umask Value = 277
```
(q) `dr-xrw--w-`

```txt
w -> 2
r -> 4
x -> 1

rx -> 5

umask Value = 215
```

#### Commands

```sh
	su - natasha
	
	vim .bash_profile
	source .bash_profile
```
> in .`bash_profile` find User specific environment and set to `umask 277`
```sh
	touch f1
	mkdir d1
	
	ll
```

![[Recording 20241111150347.webm]]

# Set the Password Expire Date

> The password for all new users in `serverb.lab.example.com` should expires after <mark style="background: #FFF3A3A6;">20 days.</mark>

#### Commands
Log out of account
```sh
	vim /etc/login.defs
```
> Find  `PASS_MAX_DAYS` change to `20`


# Assign Sudo Privilege 

>Assign the Sudo Privilege for Group "admin" and Group members can administrate without any password.

#### Commands
```sh
	vim /etc/sudoers
```
> Find `%wheel All=(ALL) All` and un-comment 
> change it to -> `%admin ALL=(ALL) NOPASSWD=ALL`

Save n exit

# Custom Welcome message on login

#### Conditions
> Configure the application RHCSA as an alies user, When login it will show the message
> `"Welcome to Advantage Pro"`

#### Commands
1) 
   ```sh
	su - alies
	vim .bash_profile
```
2) 
   > Add the following line **( you have to pass it through the variable )**
```vim
	RHCSA="Welcome to Advantage Pro" 
	echo $RHCSA
```
3) 
   ```sh
 	source .bash_profile
```

# Create a Script File

#### Conditions

1) Create a <mark style="background: #FFF3A3A6;">myfind</mark> script file ( name is `myfind `) under <mark style="background: #FFF3A3A6;">/usr/local/bin</mark> to locate files under <mark style="background: #FFF3A3A6;">/usr/share</mark> directory having size less than 1M with UID or GID .

2) After executing the <mark style="background: #FFF3A3A6;">myfind</mark> script file and listed(searched) files output has to be stored in <mark style="background: #FFF3A3A6;">/root/myfiles</mark>.


#### Commands
```sh
	vim /usr/local/bin/myfind
```

> add `find /usr/share -type f -size -1M > /root/myfiles`

```sh
	chmod +x /usr/local/bin/myfind

	myfind
```

![[Recording 20241112124322.webm]]

- [ ] If size is greater than 400k and less than 800k -> `find /usr/share -type f -size +400k   -size -800k > /root/myfiles` ![[Recording 20241112124808.webm]]
- [ ] To Add `GID` or `UID` -> `find /usr/share -type f -perm /g+s` or `find /usr/share -type f -perm /u+s`



# Boot Resetting
#servera
```sh
	lab start boot-resetting
```
> Reboot system with `CTRL` + `ALT` + `DEL` or  the reboot button

1) Enter Grub (Select `RHEL Linux Rescue` and press E)
2) Move cursor to 4th line at the end and type `rd.break` at `check net.ifnames=0 `
3) `CTRL` + `X` to start
4) Press enter to enter the maintenance mode
5) 
```sh
   mount -o remount,rw /sysroot
   chroot /sysroot
   
   passwd --stdin root
   #Changing password for user.root
   northate

	touch /.autorelabel
	vim /etc/ssh/sshd_config
```
6) **look for PermitRootLogin -> say yes**
```vim
	PermitRootLogin yes

```
7) <mark style="background: #FFF3A3A6;">exit and exit</mark> from <mark style="background: #FFF3A3A6;">chroot</mark> and <mark style="background: #FFF3A3A6;">maintenance</mark> mode. Reboot

![[Recording 20241112144704.webm]]


# Create swap

> Create a swap partition 512MB size.

#### Commands
```sh
	lsblk
	fdisk /dev/vdb
	> n
	> 
	> 
	> 
	> p
	> +512
	> t
	> 82 or swap
	> w
	> 
	
	mkswap /dev/vdb1
	vim /etc/fstab
	# /dev/vdb1 swap swap default 0 0
	swapon -a
```

![[Recording 20241112145817.webm]]


# LVM config

> Create one logical volume named database and it should be on datastore volume group with size 50 extent and assign the filesystem as ext3.
	> (i) the datastore volume group extend should be 8MiB. 
	> (ii)mount the logical volume under mount point /database.
> 
> Resize the logical volume size of 100 extent on /database directory.
> 
> Set the recommend tuned profile for your system.

![[Drawing 2024-11-17 14.05.35.excalidraw]]
#### Commands

```sh
	lsblk
	
	fdisk /dev/vdc
	> n
	> 
	> 
	> 2G
	> t
	> 8e or lv
	> w
	
	vgcreate -s 8M datastore /dev/vdc1
```
```sh
	lvcreate -l 50 -n database datastore

	lsblk

	mkfs.ext3 /dev/datastore/database

	mkdir /education
	vim /etc/fstab
	#/dev/datastore/database /education   ext3 defaults 0 0
	
	mount -a
	
	lsblk
```


![[Recording 20241112151010.webm]]

## Fell asleep, take the rest of notes for lvm and tuned from voice notes.

![[Recording 20241112151555.webm]]

# Resize / Extend LV

Resize the Linux volume to size btw 200Mib and 300Mib
```sh
	df -hT
	lvresize -L 250M /dev/vg/lv -r /dev/mapper/vg-lv
```

# Tuned

To Check Active Tuning Profile and set it to the recommended : 
```sh
	dnf install tuned
	systemctl enable --now tuned
	tuned-adm active
	tuned-adm list
	tuned-adm recommend
	tuned-adm active

	tuned-adm off
```

```button
name Press ME
type link
action https://github.com/greeenboi
customColor #C69E2D
customTextColor #fff
```
^button-69420