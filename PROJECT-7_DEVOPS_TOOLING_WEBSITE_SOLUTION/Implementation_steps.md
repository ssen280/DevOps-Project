
#### STEP 1 – PREPARE NFS SERVER

##### We will spin up a new EC2 instance with RHEL Linux 8 Operating System.
##### Based on our LVM experience from Project 6, We will configure LVM on the Server. Instead of formating the disks as ext4 you will have to format them as xfs
##### We will ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs.

##### We will create mount points on /mnt directory for the logical volumes as follow:
##### Mount lv-apps on /mnt/apps – To be used by webservers
##### Mount lv-logs on /mnt/logs – To be used by webserver logs
##### Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

##### Install NFS server, configure it to start on reboot and make sure it is u and running
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

##### we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

##### Configure access to NFS for clients within the same subnet. We will use same subet for all servers to provide relevent sevvice related access and to limit the access so that only ip addrss belong to allowed subnet group can access services ( NFS/DATABASE(MYSQL SERVER/CLIENT)/WEB(HTTPD)

##### We will check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

<img width="770" alt="Screenshot 2022-07-10 at 7 54 09 PM" src="https://user-images.githubusercontent.com/105562242/178576064-e9de2c6a-2be7-4971-9eec-0e55335b6e91.png">

<img width="1477" alt="Screenshot 2022-07-10 at 7 55 44 PM" src="https://user-images.githubusercontent.com/105562242/178576106-2fd615ae-bd0a-4e1c-b64c-3717e3a02d25.png">

<img width="929" alt="Screenshot 2022-07-10 at 7 58 33 PM" src="https://user-images.githubusercontent.com/105562242/178576150-841b54e7-804f-4dd6-b36e-1f700d85a95e.png">

<img width="818" alt="Screenshot 2022-07-10 at 8 03 24 PM" src="https://user-images.githubusercontent.com/105562242/178576277-51db0dd5-e492-48f3-a63a-804dd8dba6d2.png">

<img width="855" alt="Screenshot 2022-07-10 at 8 04 44 PM" src="https://user-images.githubusercontent.com/105562242/178576317-1d7ac143-130f-4544-9059-451f635ee1b8.png">

<img width="1424" alt="Screenshot 2022-07-10 at 8 08 58 PM" src="https://user-images.githubusercontent.com/105562242/178576373-a4380db8-f970-47c2-ad6b-87804d8b5edb.png">

<img width="1020" alt="Screenshot 2022-07-10 at 8 09 22 PM" src="https://user-images.githubusercontent.com/105562242/178576432-6f1a50c2-4014-4f09-86bf-52889a603a0c.png">

<img width="1338" alt="Screenshot 2022-07-10 at 8 12 26 PM" src="https://user-images.githubusercontent.com/105562242/178576506-28fd5cac-d67e-4d7e-ad13-9ffa5042377e.png">


<img width="1369" alt="Screenshot 2022-07-11 at 8 09 38 PM" src="https://user-images.githubusercontent.com/105562242/178576628-199a3782-92d6-4bfc-9614-4b58bb2e8f1e.png">

<img width="1279" alt="Screenshot 2022-07-11 at 8 15 08 PM" src="https://user-images.githubusercontent.com/105562242/178576683-57897d06-46f0-45a8-bf6d-59638d21e715.png">

<img width="1727" alt="Screenshot 2022-07-11 at 8 24 04 PM" src="https://user-images.githubusercontent.com/105562242/178577271-50acb53f-937f-4e3d-bf29-1fe7a4e01d70.png">

<img width="884" alt="Screenshot 2022-07-13 at 1 01 42 AM" src="https://user-images.githubusercontent.com/105562242/178578580-98f94923-9f0b-4e9b-a3d3-f5aa04534b49.png">

<img width="892" alt="Screenshot 2022-07-13 at 1 14 18 AM" src="https://user-images.githubusercontent.com/105562242/178580881-e7a3da5a-e4ad-4569-9754-4270b79db94c.png">



#### We will open below ports to respective servers security groups:
##### TCP 111, UDP 111, UDP 2049 ports on NFS server so that other servers can access shared folder. 
##### 3306 port on database server so that mysql client can communication to database server. 
##### 80 port on web server so that httpd can access port 80 for webpage


#### STEP 2 — CONFIGURE THE DATABASE SERVER

##### We will install mysql server and check service status

<img width="892" alt="Screenshot 2022-07-13 at 1 27 19 AM" src="https://user-images.githubusercontent.com/105562242/178583116-046b79ed-e362-4757-b83b-79c71b095433.png">

##### We will create a database and name it tooling and we will create user webaccess user on tooling database to do anything only from the webservers subnet cidr

<img width="1323" alt="Screenshot 2022-07-11 at 10 49 00 PM" src="https://user-images.githubusercontent.com/105562242/178584130-c3e2e147-3ae5-4605-9c40-30544b231453.png">

#### Step 3 — Prepare the Web Servers

##### We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database. We already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

##### This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

##### During the next steps we will do following:

Configure NFS client (this step must be done on all three servers)
Deploy a Tooling application to our Web Servers into a shared NFS folder
Configure the Web Servers to work with a single MySQL database
Launch a new EC2 instance with RHEL 8 Operating System
Install NFS client

` sudo yum install nfs-utils nfs4-acl-tools -y `

Mount /var/www/ and target the NFS server’s export for apps
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:

```
sudo vi /etc/fstab
```
add following line

` <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0 `

Install Remi’s repository, Apache and PHP

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1

```
Repeat above steps for another 2 Web Servers.

<img width="886" alt="Screenshot 2022-07-13 at 1 49 27 AM" src="https://user-images.githubusercontent.com/105562242/178587481-b6f44556-6d06-4997-84d8-c81dfde2a82e.png">

<img width="889" alt="Screenshot 2022-07-13 at 1 51 16 AM" src="https://user-images.githubusercontent.com/105562242/178587754-ade68639-442f-4556-990e-091bdbacf5c1.png">

<img width="890" alt="Screenshot 2022-07-13 at 1 52 01 AM" src="https://user-images.githubusercontent.com/105562242/178587831-fe58e576-e4ec-47a8-b0ec-d477f6e3a971.png">

<img width="891" alt="Screenshot 2022-07-13 at 1 53 09 AM" src="https://user-images.githubusercontent.com/105562242/178587999-edfae1b2-d95e-466a-85dc-12c5f460695c.png">

Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.

<img width="888" alt="Screenshot 2022-07-13 at 2 00 52 AM" src="https://user-images.githubusercontent.com/105562242/178589674-ac4464f7-05a8-4b02-b0b5-0b1ee5ba2900.png">


