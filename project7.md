## Documentation for Project 7 (DevOps Website Tooling)

### Entire View of partitioned Volume 
`lsblk`
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```
![xvdf](./Images-NfsServer/nfs-xvdf.png)
![xvdg](./Images-NfsServer/nfs-xvdg.png)
![xvdh](./Images-NfsServer/nfx-xvdh.png)
![list-of-newly-partitioned-volume](./Images-NfsServer/nfs-partioned-disk.png)

### LVM2 Installation
`sudo yum install lvm2`
`sudo lvmdiskscan`
![lVM2-Installation](./Images-NfsServer/install-lvm.png)
![lVM2-Installation](./Images-NfsServer/lvmdiskscan.png)

### Partition Marked as Physical Volume
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`
`sudo pvs`
![Marking-partition-as-physical-Volume](./Images-NfsServer/nfs-physical-volumes.png)
![Marking-partition-as-physical-Volume](./Images-NfsServer/nfs-pvs.png)

### Creating a Volume Group
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
`sudo vgs`
![Volume-Group-Creation](./Images-NfsServer/nfs-volume-group.png)

### Creating Logical Volume
`sudo lvcreate -n lv-apps -L 9G webdata-vg`
`sudo lvcreate -n lv-logs -L 9G webdata-vg`
`sudo lvcreate -n lv-opt -L 9G webdata-vg`
`sudo lvs`
![Logical-Volume-Creation](./Images-NfsServer/nfs-logical-volumes.png)

### Formatting Logical Volume as xfs
`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`
![Formatting-Logical-Volume](./Images-NfsServer/formatting-logical-volumes.png)

### Mounting our logical volumes on their respective directories
`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`
![Mounting-logical-volumes](./Images-NfsServer/nfs-mount.png)

## Preparing NFS Server
`sudo yum -y update`
`sudo yum install nfs-utils -y`
![NFS-Server-Installation](./Images-NfsServer/nfs-update.png)
![NFS-Server-Installation](./Images-NfsServer/install-lvm.png)

### NFS Server Running
`sudo systemctl start nfs-server.service`
`sudo systemctl enable nfs-server.service`
`sudo systemctl status nfs-server.service`
![NFS-Server-Running](./Images-NfsServer/nfs-status.png)

## Setting Read and Write Permission on NFS Server for Webservers to read and write on NFS
`sudo chown -R nobody: /mnt/apps`
`sudo chown -R nobody: /mnt/logs`
`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`
`sudo chmod -R 777 /mnt/logs`
`sudo chmod -R 777 /mnt/opt`

![setting-read-and-write-permissions-for-webservers-on-NFSSERVER.](./Images-NfsServer/nfs-change-ownership.png)

## Exporting our mounts for access by clients within the same subnet cidr
`sudo vi /etc/exports`
`sudo exportfs -arv`
![exporting-our-mounts](./Images-NfsServer/nfs-subnet.png)

## Opening TCP and UDP Ports for access by webservers under security Group
![Security-Group-Configuration](./Images-NfsServer/ports.png)

## **Configuring Database Server**
`sudo apt update`
`sudo apt install mysql-server`
![mysql-server-installed](./Images-DB/db-mysql.png)

### Creating Database tooling and User
`create database tooling;`
`create user 'webaccess'@'172.31.80.0/20' identified by 'mypass';`
`grant all privileges on tooling.* to 'webaccess'@'172.31.80.0/20';`
`flush privileges;`
![creating-tooling-database-and-user](./Images-DB/db-create-user.png)

### Installing NFS Client on Webserver
`sudo yum install nfs-utils nfs4-acl-tools -y`

![Installing-NFS-Client](./Images-Webserver/web1-install-nfs-client.png)
![Installing-NFS-Client](./Images-Webserver/web2&3-nfs-client-installation.png)

### Connecting to tooling database from Webserver
`mysql -h 172.31.92.231 -u webaccess -p tooling < tooling-db.sql`

![connecting-to-tooling-Database](./Images-Webserver/web-connect-db.png)

## Mounting /var/www/ and target the NFS server’s export for apps

### Make sure that the changes will persist on Web Server after reboot:

`sudo mkdir /var/www`
`sudo mount -t nfs -o rw,nosuid 172.31.93.135:/mnt/apps /var/www`
`sudo vi /etc/fstab`
`172.31.93.135:/mnt/apps /var/www nfs defaults 0 0`

![Mounting-server-export](./Images-Webserver/web-target-nfsserver.png)

### Installing Remi’s repository, Apache and PHP
`sudo yum install httpd -y`

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo dnf module reset php`

`sudo dnf module enable php:remi-7.4`

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo setsebool -P httpd_execmem 1`

![connecting-to-tooling-Database](./Images-Webserver/web-installing-apache.png)
![connecting-to-tooling-Database](./Images-Webserver/web-installing-dnf-utils.png)
![connecting-to-tooling-Database](./Images-Webserver/web-installing-php-dependecies.png)
![connecting-to-tooling-Database](./Images-Webserver/web-installing-php-dependiecies&enable.png)
![connecting-to-tooling-Database](./Images-Webserver/web-remi-repo.png)

### Installing git to allow forking of website
`sudo yum install git`

![Installing-git](./Images-Webserver/web-install-git.png)

### cloning tooling repo from git account
`git init`
`git clone https://github.com/darey-io/tooling.git`

![Tooling-repo-cloned](./Images-Webserver/web-gitclone-toolings.png)

### Deploying html folder from repo to /var/www/html
`sudo cp -R html/. /var/www/html`

![HTML-folder-from-repo-deployed-to-var-www-html](./Images-Webserver/web-copy-toolings.png)

### Accessing the tooling database forked from github from Database

![tooling-db-accessed-from-database](./Images-DB/mysql-db-toolings.png)

### Tooling Website Live

![Tooling-website-live](./Images-NfsServer/loginpage.png)

### Logged in with admin info

![logged-in-with-admin-info](./Images-NfsServer/successpage.png)