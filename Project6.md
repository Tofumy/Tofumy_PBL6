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

