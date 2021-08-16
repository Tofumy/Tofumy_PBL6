## WEB SOLUTION WITH WORDPRESS

Project 6 consists of two parts:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution

### Requirements

The 3-Tier Setup
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server



### Step 1 - Preparing the Web Server

- Created an EC2 instance in the AWS that will server as Web Server (We are using Red Hat OS for the linux image)
- Created 3 EBS (Elastic Block Store) Volume and attach to the Web Server
    - First we confirmed the availability zone where the 2 EC2 instances are created "us-east-2a", that will be the same AZ that           would be chosen for the EBS volumes

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/create-volume.PNG)

- We attached each of the EBS volume to the web server

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/attach-volume.PNG)

- Connected to the Linux Web Server and ran the below cmd to confirm the volumes are attached and are showing

`lsblk`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lsblk.PNG)

- We then use the `df -h` to check the free space in the server and also the mount points available
- We then use the 'gdisk' cmd to create a partition to each volume

`sudo gdisk /dev/xvdf`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/gdisk-xvdf.PNG)

`sudo gdisk /dev/xvdg`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/gdisk-xvdg.PNG)

`sudo gdisk /dev/xvdh`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/gdisk-xvdh.PNG)

ran the `lsblk` to confirm again

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/new-lsblk.PNG)


- So we need to install the LVM2 to create logical volumes

`sudo yum install lvm2`

- We ran the `sudo lvmdiskscan` to confirm the available partitions, we could see 4 partitions but no physical volumes created yet.

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lvmdiskscan.PNG)

- We then created physical volumes on the partitions using the below:

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

we then confirmed the PVs have been created using `sudo pvs`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/pvcreate.PNG)

- We have to add all the PVs to a volume group (VG), we do that using the below and the name should be "webdata-vg"

`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/vgcreate.PNG)


- Use the "lvcreate" utility to create 2 LV (apps-lv and apps-lv)
    - apps-lv : for storing data for the Website
    - logs-lv : for storing data for logs

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

to confirm the LVs
`sudo lvs`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lvcreate-webdata-vg.PNG)

- We then ran this `sudo vgdisplay -v` to confirm the setup for the VG, PV, LV

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/vgdisplay.PNG)

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lsblk-vg.PNG)

- We used the mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/mkfs-ext4.PNG)

- Created ext /var/www/html directory ext to store website files

`sudo mkdir -p /var/www/html`

- Created ext /home/recovery/logs ext to store backup of log data

`sudo mkdir -p /home/recovery/logs`

- Mounted /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

- Used the rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before     mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/mount-apps-lv.PNG)

- Mounted /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

- Restored the log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/log/. /var/log`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/mount-logs-lv.PNG)

- Updated the /etc/fstab file so that the mount configuration will persist after restart of the server.
- We had to look for the UUID of the 2 LV of the "apps-lv" and the "logs-lv" using this `sudo blkid`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/blkid.PNG)

Took note of the below information to add to this file /etc/fstab

UUID=5287c731-e3c8-4f5a-98f8-0ca188b03222 /var/www/html ext4 defaults 0 0
UUID=7989a97e-bc63-494d-ad15-654b79af59b1 /var/log      ext4 defaults 0 0 

`sudo vi /etc/fstab`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/sudo-vi-fstab.PNG)

- Tested the configuration and reloaded the daemon

 `sudo mount -a`
 
 `sudo systemctl daemon-reload`
 
- Verified the setup by running `df -h`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/df-h.PNG)



### Step 2 - Preparing the Database Server

Now for the DB Server, we created 3 EBS volume in the same AZ as the Database Sever which is "us-east-2a"

- We attached each of the EBS volume to the DB server


- Connected to the Linux DB Server and ran the below cmd to confirm the volumes are attached and are showing

`lsblk`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lsblk2.PNG)

- We then use the `df -h` to check the free space in the server and also the mount points available
- We then use the 'gdisk' cmd to create a partition to each volume

`sudo gdisk /dev/xvdf`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/gdisk-xvdf2.PNG)

`sudo gdisk /dev/xvdg`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/gdisk-xvdg2.PNG)

`sudo gdisk /dev/xvdh`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/gdisk-xvdh2.PNG)

ran the `lsblk` to confirm again

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/new-lsblk2.PNG)


- So we need to install the LVM2 to create logical volumes

`sudo yum install lvm2`

- We ran the `sudo lvmdiskscan` to confirm the available partitions, we could see 4 partitions but no physical volumes created yet.

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lvmdiskscan2.PNG)

- We then created physical volumes on the partitions using the below:

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

we then confirmed the PVs have been created using `sudo pvs`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/pvcreate2.PNG)

- We have to add all the PVs to a volume group (VG), we do that using the below and the name should be "webdata-vg"

`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/vgcreate2.PNG)

- We use `sudo vgs` to confirm the volume group has been created


- Use the "lvcreate" utility to create 1 LV (db-lv)
    - db-lv : for storing data for the db

`sudo lvcreate -n db-lv -L 20G dbdata-vg`

to confirm the LVs
`sudo lvs`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lvcreate-dbdata-vg.PNG)

- We then ran this `sudo vgdisplay -v` to confirm the setup for the VG, PV, LV

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/vgdisplay2.PNG)

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/lsblk-vg2.PNG)

- We used the mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/db-lv` OR  `sudo mkfs.ext4 /dev/webdata-vg/db-lv`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/mkfs-ext4-2.PNG)

- Created ext /db directory ext to store db files

`sudo mkdir /db`

NB: Checking the directory /db using `sudo ls -l /db` shows it is empty, so no need to backup

- Mounted /db on db-lv logical volume

`sudo mount /dev/dbdata-vg/db-lv /db`


- Updated the /etc/fstab file so that the mount configuration will persist after restart of the server.
- We had to look for the UUID of the LV of the "db-lv" using this `sudo blkid`


Took note of the below information to add to this file /etc/fstab

UUID=c5759bf6-89c6-4914-8768-25c13efb7101  /db ext4 defaults 0 0


`sudo vi /etc/fstab`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/sudo-vi-fstab2.PNG)

- Tested the configuration and reloaded the daemon

 `sudo mount -a`
 
 `sudo systemctl daemon-reload`
 
- Verified the setup by running `df -h`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/df-h2.PNG)



### Step 3 — Install WordPress on your Web Server EC2


- Updated the repository using `sudo yum -y update`

- Installed wget, Apache and it’s dependencies using `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

- Started Apache

`sudo systemctl enable httpd`

`sudo systemctl start httpd`

To install PHP and it’s dependencies

``` bash

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1

```

Restarted Apache

`sudo systemctl restart httpd`

Downloaded wordpress 

  `mkdir wordpress`
  
  `cd   wordpress`
  
  `sudo wget http://wordpress.org/latest.tar.gz`
  
  ![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/wordpress.PNG)
  
Extracted the compressed word press installation file amnd then removed the compressed file
  
  `sudo tar xzvf latest.tar.gz`
  
  `sudo rm -rf latest.tar.gz`
  
Changed the name of the config file to "wp-config.php"  and then copied the file to ext /var/www/html/ ext path
  
  `cp wordpress/wp-config-sample.php wordpress/wp-config.php`
  
  `cp -R wordpress /var/www/html/`
  
Configured SELinux Policies

  `sudo chown -R apache:apache /var/www/html/wordpress`
  
  `sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`
  
  `sudo setsebool -P httpd_can_network_connect=1`
  
  
  
  ### Step 4 — Install MySQL on your DB Server EC2
  
  
  `sudo yum update`
  
  `sudo yum install mysql-server`
  
![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/install-mysql.PNG)
  
Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

`sudo systemctl restart mysqld`
`sudo systemctl enable mysqld`

![screenshot](https://github.com/Tofumy/Tofumy_PBL6/blob/main/systemctl-mysqld.PNG)

