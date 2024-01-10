 # Project 7:  Devops Tooling Website Solution

 ### In this project, it consists of following components:

1. Infrastructure: AWS

2. Webserver Linux: Red Hat Enterprise Linux 8. For Rhel 8 server i used this 
ami RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 (ami-035c5dc086849b5de)

3. Database Server: Ubuntu  20.04 + MySQL

4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server

5. Programming Language: PHP

6. Code Repository: GitHub

## Below are the steps I took to complete this project

### Step 1 - Prepare NFS Server

1. Spin up a new EC2 instance with RHEL Linux 8 Operating System.

![instance](./images/1.PNG)

2. Configure LVM on the Server

a. Created 3 volumes in the same AZ as my Web Server EC2, each of 10 GiB and attached all three volumes one by one to my Web Server EC2 instance

![instance](./images/1b.PNG)

b. Opened up the Linux terminal to begin configuration

![instance](./images/1c.PNG)

c. Used `lsblk` command to inspect what block devices are attached to the server and names of my newly created devices

![instance](./images/1d.PNG)

Note that all devices in Linux reside in /dev/ directory. I inspected it with `ls /dev/` and made sure I see all 3 newly created block devices there - their names are xvdf, xvdh, xvdg.

![instance](./images/1e.PNG)

d. Used `df -h` command to see all mounts and free space on my server

![instance](./images/1f.PNG)

e. Used gdisk utility to create a single partition on each of the 3 disks
`sudo gdisk /dev/xvdf` On the prompts i entered n followed by p and w. I did this step on each of the disks, xvdf,xvdg,xvdh

![instance](./images/1ga.PNG)
![instance](./images/1gb.PNG)
![instance](./images/1gc.PNG)

f. Used `lsblk` utility to view the newly configured partition on each of the 3 disks.

![instance](./images/1h.PNG)

g. I installed lvm2 package using `sudo yum install lvm2`

![instance](./images/1ia.PNG)
![instance](./images/1ib.PNG)

h. I ran `sudo lvmdiskscan` command to check for available partitions.

![instance](./images/1j.PNG)

i. I used pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

![instance](./images/1k.PNG)

j. I verified that my Physical volume has been created successfully by running `sudo pvs`

![instance](./images/1l.PNG)

k. I used vgcreate utility to add all 3 PVs to a volume group (VG). And named the VG webdata-vg. 
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![instance](./images/1m.PNG)

l. Verified that my VG has been created successfully by running `sudo vgs`

![instance](./images/1n.PNG)

m. Used lvcreate utility to create 3 logical volumes, lv-apps, lv-logs, and lv-opt
`sudo lvcreate -n lv-apps -L 9G webdata-vg`
`sudo lvcreate -n lv-logs -L 9G webdata-vg`
`sudo lvcreate -n lv-opt -L 9G webdata-vg`

![instance](./images/1o.PNG)

n. I verified that my Logical Volume has been created successfully by running `sudo lvs`

![instance](./images/1p.PNG)

o. Verified the entire setup with `sudo vgdisplay -v`

![instance](./images/1qa.PNG)
![instance](./images/1qb.PNG)
![instance](./images/1qc.PNG)
![instance](./images/1qd.PNG)
![instance](./images/1qe.PNG)

p. Used `sudo lsblk` command to inspect what block devices are attached to the server

![instance](./images/1r.PNG)

q. Use mkfs.xfs to format the logical volumes with xfs filesystem
`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![instance](./images/1sa.PNG)
![instance](./images/1sb.PNG)

r. Created /mnt/apps,/mnt/logs, /mnt/opt  directories to store website files, log files etc
`sudo mkdir -p /mnt/apps`
`sudo mkdir -p /mnt/logs`
`sudo mkdir -p /mnt/opt`

![instance](./images/2a.PNG)

s. Created mount points on /mnt directory for the logical volumes as follow:
Mounted lv-apps on /mnt/apps  - To be used by webservers
Mount lv-logs on  /mnt/logs - To be used by webserver logs
Mount lv-opt  on  /mnt/opt  - To be used by Jenkins server in Project 8

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
`sudo mount /dev/webdata-vg/lv-opt /mnt/opt/`
`sudo mount /dev/webdata-vg/lv-logs /mntlongs/`

![instance](./images/2b.PNG)
![instance](./images/2c.PNG)

3. Install NFS server, configure it to start on reboot and make sure it is u and running

`sudo yum -y update`

![instance](./images/4a.PNG)

`sudo yum install nfs-utils -y`

![instance](./images/4bi.PNG)
![instance](./images/4bii.PNG)

`sudo systemctl start nfs-server.service`   `sudo systemctl enable nfs-server.service`   `sudo systemctl status nfs-server.service`

![instance](./images/4c.PNG)

4. Exported the mounts for webservers' subnet cidr to connect as clients. For simplicity, I installed all my three Web Servers inside the same subnet.
And this is how I checked my subnet cidr - I opened my EC2 details in AWS web console and located 'Networking' tab and open a Subnet link.

a. I made sure I set up permission that will allow my Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps`
`sudo chown -R nobody: /mnt/logs`
`sudo chown -R nobody: /mnt/opt`
`sudo chmod -R 777 /mnt/apps`
`sudo chmod -R 777 /mnt/logs`
`sudo chmod -R 777 /mnt/opt`

![instance](./images/5.PNG)

`sudo systemctl restart nfs-server.service`

![instance](./images/6.PNG)

b. Configured access to NFS for clients within the same subnet 

`sudo vi /etc/exports`

![instance](./images/7a.PNG)
![instance](./images/7b.PNG)

`sudo exportfs -arv`

![instance](./images/7c.PNG)

5. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`sudo rpcinfo -p | grep nfs`

![instance](./images/8.PNG)

![instance](./images/xy.PNG)

### Step 2 — Configure the database server

a. `sudo apt update`

![instance](./images/10a.PNG)
![instance](./images/10b.PNG)

b. Install MySQL server

`sudo apt install mysql-server`

![instance](./images/11a.PNG)
![instance](./images/11b.PNG)

c. Created a database and named it tooling, webaccess as the user, and grant ed permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

`CREATE DATABSE tooling;`
`create user `webaccess`@`172.31.32.0/20` IDENTIFIED BY 'Password';`
`GRANT ALL ON tooling.* TO 'webaccess'@'172.31.32.0/20';`
`FLUSH PRIVILEGES;`
`SHOW DATABASES;`

![instance](./images/12.PNG)

### Step 3 — Prepare the Web Servers

Durind this step I did the following:

i. Configured NFS client (step must be done on all three servers)
ii. Deployed a Tooling application to our Web Servers into a shared NFS folder
iii. Configured the Web Servers to work with a single MySQL database

1. Launched a new EC2 instance with RHEL 8 Operating System

2. Installed NFS client
`sudo yum install nfs-utils nfs4-acl-tools -y`

![instance](./images/13a.PNG)
![instance](./images/13b.PNG)

3. Mount /var/www/ and target the NFS server's export for apps. 'sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www'

`sudo mkdir /var/www`    `sudo mount -t nfs -o rw,nosuid 172.31.34.162:/mnt/apps /var/www`

![instance](./images/15.PNG)

4. Verify that NFS was mounted successfully by running `df -h` 

![instance](./images/15.PNG)

5. To make sure that the changes will persist on Web Server after reboot I ran `sudo vi /etc/fstab` to edit the file and add following line '<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0'
`172.31.34.162:/mnt/apps /var/www nfs defaults 0 0`

![instance](./images/16a.PNG)
![instance](./images/16b.PNG)

6. Install Remi's repository, Apache and PHP

`sudo yum install httpd -y`

![instance](./images/17a.PNG)
![instance](./images/17b.PNG)

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![instance](./images/18a.PNG)
![instance](./images/18b.PNG)

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![instance](./images/19a.PNG)
![instance](./images/19b.PNG)

`sudo dnf module reset php`

![instance](./images/20.PNG)

`sudo dnf module enable php:remi-7.4`

![instance](./images/21.PNG)

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![instance](./images/22a.PNG)
![instance](./images/22b.PNG)

`sudo systemctl start php-fpm`

![instance](./images/23.PNG)

`sudo systemctl enable php-fpm`

![instance](./images/24.PNG)

`sudo setsebool -P httpd_execmem 1`

![instance](./images/25.PNG)

7. I verified that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. I saw the same files which means NFS was mounted correctly.

On web server, I ran `ls /var/www`

![instance](./images/26a.PNG)

On NFS server, I ran `ls /mnt/apps`

![instance](./images/26b.PNG)

I went ahead to create a new file touch test.txt on the web server and check if the same file is accessible from the NFS Server.

On web server

![instance](./images/26c.PNG)

On NFS server

![instance](./images/26d.PNG)

8. I located the log folder for Apache on the Web Server and mounted it to NFS server's export for logs.

![instance](./images/27.PNG)

9. I mounted the /mnt/logs to /var/log/httpd.
'sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd'

`sudo mount -t nfs -o rw,nosuid 172.31.34.162:/mnt/logs /var/log/httpd`

![instance](./images/27b.PNG)

10. To verify the setup
`df -h`

![instance](./images/27c.PNG)

11. To make sure the mount point will persist after reboot, I ran `sudo vi /etc/fstab` to edit the file and add '<NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd nfs defaults 0 0'
`172.31.34.162:/mnt/logs /var/log/httpd nfs defaults 0 0`

![instance](./images/27d.PNG)
![instance](./images/27e.PNG)

12. Forked the tooling source code from Darey.io Github Account to my Github account.

I did not have git installed in my system so i took the following step

`sudo yum install git`

![instance](./images/28a.PNG)

Then, cloned the repository into my server

`git clone <the repo link>`

![instance](./images/28b.PNG)

13. Deployed the tooling website's code to the Webserver. Ensured that the html folder from the repository was deployed to /var/www/html

![instance](./images/29.PNG)

14. To make this change permanent - i opened the following config file `sudo vi /etc/sysconfig/selinux` and set SELINUX=disabled, then restrt httpd.

![instance](./images/30a.PNG)
![instance](./images/30b.PNG)

15. Change the bind address `sudo vi /etc//mysql/mysql.conf.d/mysqld.cnf`

![instance](./images/31a.PNG)
![instance](./images/31b.PNG)

16. Updated the website's configuration to connect to the database in /var/www/html/functions.php file by cd into the location of the tooling folder in my server and applied the tooling-db.sql script to my database 
`sudo vi /var/www/html/functions.php`

![instance](./images/33a.PNG)
![instance](./images/33b.PNG)

17. On the web server install the mysql
`sudo yum install mysql -y`

![instance](./images/34a.PNG)
![instance](./images/34b.PNG)

18. Applied tooling-db.sql script to my database using this command mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
`sudo mysql -h 172.31.36.84 -u webaccess -p tooling < tooling-db.sql`

![instance](./images/35.PNG)

19. I logged into the mysql to see the database I created earlier and to see the user and the password

![instance](./images/36a.PNG)
![instance](./images/36b.PNG)

20. I decrypted the password with this site

![instance](./images/37a.PNG)
![instance](./images/37b.PNG)


21. Opened the website in my browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php

![instance](./images/38.PNG)
![instance](./images/39.PNG)






















































































