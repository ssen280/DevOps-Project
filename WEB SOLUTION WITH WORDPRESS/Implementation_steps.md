
#### Prject - 6

#### In this project we will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

#### This project consists of two parts:

##### Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give practical experience of working with disks, partitions and volumes in Linux.

##### Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify skills of deploying Web and DB tiers of Web solution.

#### Three-tier Architecture :

##### Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

##### Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

<img width="1044" alt="Screenshot 2022-05-19 at 6 28 27 PM" src="https://user-images.githubusercontent.com/105562242/169298939-515957c1-904d-469c-9562-e5d1067e3d03.png">

##### Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
##### Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
##### Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

##### In this project, we will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.

##### 3-Tier Setup : 
* A Laptop or PC to serve as a client
* An EC2 Linux Server as a web server (This is where you will install WordPress)
* An EC2 Linux server as a database (DB) server( RedHat OS )

#### Step 1 — Prepare a Web Server 
 * Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
 * Attach all three volumes one by one to your Web Server EC2 instance
 * Open up the Linux terminal to begin configuration
 * Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
 * Use df -h command to see all mounts and free space on your server
 * Use gdisk utility to create a single partition on each of the 3 disks
 * ``` sudo gdisk /dev/xvdf ```
 * Use lsblk utility to view the newly configured partition on each of the 3 disks.
 * Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
 * Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
 ```
 sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

```
 * Verify that your Physical volume has been created successfully by running sudo pvs
 * Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
 ```
 sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
 ```
 * Verify that your VG has been created successfully by running sudo vgs
 * Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
 ```
 sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
 * Verify that your Logical Volume has been created successfully by running sudo lvs
 * Verify the entire setup
 ```
 sudo vgdisplay -v #view complete setup - VG, PV, and LV
 sudo lsblk 
 ```
 <img width="888" alt="Screenshot 2022-05-19 at 6 53 26 PM" src="https://user-images.githubusercontent.com/105562242/169303703-066b48f8-c235-41b7-b650-fe3c19cb77b1.png">
 
 * Use mkfs.ext4 to format the logical volumes with ext4 filesystem
 ```
 sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
 * Create /var/www/html directory to store website files
 ```
 sudo mkdir -p /var/www/html
 ```
 * Create /home/recovery/logs to store backup of log data
 ```
 sudo mkdir -p /home/recovery/logs
 ```
 * Mount /var/www/html on apps-lv logical volume
 ```
 sudo mount /dev/webdata-vg/apps-lv /var/www/html/
 ```
 * Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
 ```
 sudo rsync -av /var/log/. /home/recovery/logs/
 ```
 * Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)
 ```
 sudo mount /dev/webdata-vg/logs-lv /var/log
 ```
 * Restore log files back into /var/log directory
 ```
 sudo rsync -av /home/recovery/logs/. /var/log
 ```
 * Update /etc/fstab file so that the mount configuration will persist after restart of the server.
 * The UUID of the device will be used to update the /etc/fstab file
 ```
 sudo blkid
 sudo vi /etc/fstab
 ```
 * Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
 
 <img width="883" alt="Screenshot 2022-05-19 at 7 00 58 PM" src="https://user-images.githubusercontent.com/105562242/169305232-8860f37f-7302-4caf-b783-4cdc2b966436.png">

 * Test the configuration and reload the daemon
 ```
  sudo mount -a
 sudo systemctl daemon-reload
 ```
 * Verify your setup by running df -h, output must look like this:

 <img width="886" alt="Screenshot 2022-05-19 at 7 03 10 PM" src="https://user-images.githubusercontent.com/105562242/169305687-88401c7d-91ee-4fdf-9cbb-8242f509a8b5.png">
 
 #### Step 2 — Prepare the Database Server
 
##### Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
##### Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

------------------------------------
#### Refer below EC2 instances for web server and Database servers and volumes attached to them along with all command outputs to manage disk. Here we have given command out put only for database server as same commands applicable to web server as well !!!

<img width="1480" alt="Screenshot 2022-05-19 at 7 10 22 PM" src="https://user-images.githubusercontent.com/105562242/169307252-310e0a6d-66db-4f9f-bffd-1bf6435d93c4.png">

<img width="963" alt="Screenshot 2022-05-19 at 12 01 29 AM" src="https://user-images.githubusercontent.com/105562242/169307496-754ff96c-bfab-4d49-811a-a981aff738b6.png">

<img width="1450" alt="Screenshot 2022-05-19 at 12 02 02 AM" src="https://user-images.githubusercontent.com/105562242/169307564-1666b29e-6ef5-4449-9420-5a1a8d6c9320.png">

<img width="1016" alt="Screenshot 2022-05-19 at 12 02 24 AM" src="https://user-images.githubusercontent.com/105562242/169307597-ca5072a9-353e-4cf6-8f60-87d38604e996.png">

<img width="1022" alt="Screenshot 2022-05-19 at 12 04 48 AM" src="https://user-images.githubusercontent.com/105562242/169307626-982a7d1b-e66d-4b57-84d4-e9c1888c9b5f.png">

<img width="845" alt="Screenshot 2022-05-19 at 12 05 21 AM" src="https://user-images.githubusercontent.com/105562242/169307690-e0b957df-1c2b-4080-b9a6-b5aeee0f10f8.png">

<img width="1259" alt="Screenshot 2022-05-19 at 12 09 39 AM" src="https://user-images.githubusercontent.com/105562242/169307723-25c3d8f7-dc79-4dd8-91b3-5df4169d358b.png">

<img width="1210" alt="Screenshot 2022-05-19 at 12 13 01 AM" src="https://user-images.githubusercontent.com/105562242/169307744-8838d599-64cd-4b80-b96e-ce5df043c6bd.png">


<img width="1706" alt="Screenshot 2022-05-19 at 12 17 38 AM" src="https://user-images.githubusercontent.com/105562242/169307772-c1b197f4-9f79-4d85-87ae-2a74d957d7d4.png">

<img width="1320" alt="Screenshot 2022-05-19 at 12 21 39 AM" src="https://user-images.githubusercontent.com/105562242/169307805-8eecd06b-f596-4baa-b7c3-2cd5b65cae11.png">


<img width="1011" alt="Screenshot 2022-05-19 at 12 22 24 AM" src="https://user-images.githubusercontent.com/105562242/169307836-d5004946-5ea7-4f3a-a66c-020cda484ae6.png">

----------------------------------

#### Step 3 — Install WordPress on your Web Server EC2

* Update the repository
```
sudo yum -y update
```
* Install wget, Apache and it’s dependencies
```
Install wget, Apache and it’s dependencies
```
* sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
* Start Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
* To install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
* Restart Apache
```
sudo systemctl restart httpd
```
* Download wordpress and copy wordpress to var/www/html
```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
```
 * Configure SELinux Policies
 ```
   sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
 ```
 <img width="886" alt="Screenshot 2022-05-19 at 7 20 00 PM" src="https://user-images.githubusercontent.com/105562242/169309449-1fabf456-1469-42ba-a617-5868135fd51f.png">
 
 #### Step 4 — Install MySQL on your DB Server EC2
 
  ```
  sudo yum update
  sudo yum install mysql-server
```
  * Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
  ```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
  ```
  #### Step 5 — Configure DB to work with WordPress
  
  ```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `<desire_username>`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY <desire_password>';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```
 #### Step 6 — Configure WordPress to connect to remote database.
 
 <img width="1089" alt="Screenshot 2022-05-19 at 7 25 13 PM" src="https://user-images.githubusercontent.com/105562242/169310627-e8080023-1919-4366-b49c-f746623a9dda.png">

 * Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
 ```
 sudo yum install mysql
 sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
 ```
 * Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
 * Change permissions and configuration so Apache could use WordPress:
 * Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
 * Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/
  
  
<img width="1317" alt="Screenshot 2022-05-19 at 7 30 05 PM" src="https://user-images.githubusercontent.com/105562242/169311718-cca1f6ed-8daf-4cb0-8d99-d3ac6221bcd7.png">

<img width="1193" alt="Screenshot 2022-05-19 at 7 31 41 PM" src="https://user-images.githubusercontent.com/105562242/169312049-d1d1c115-eca0-4f4d-8330-88e332717a76.png">

<img width="1122" alt="Screenshot 2022-05-19 at 7 32 39 PM" src="https://user-images.githubusercontent.com/105562242/169312231-5025a6d6-1d73-4f87-b150-9da35ed1563b.png">

* Here we can see database connection error to wordpress web page.
* As we can see that we are able to connect to mysql server from web server to database server hence there is no issue with that. 
* Further we will check wp-config.php file in wordpress folder. 
* Here we can see that database details did not populate, due to that we can getting database connection error.
* We will provide database details to wp-config.php file. 
* Now we can see that we are able to access wordpress web page. 
  
<img width="1359" alt="Screenshot 2022-05-19 at 2 53 26 PM" src="https://user-images.githubusercontent.com/105562242/169313270-637c5332-d773-41af-b752-45de2985a6d1.png">
  
<img width="1440" alt="Screenshot 2022-05-19 at 2 58 42 PM" src="https://user-images.githubusercontent.com/105562242/169313369-028629a6-c292-45cc-8454-9d449c1334d2.png">

<img width="1182" alt="Screenshot 2022-05-19 at 7 38 16 PM" src="https://user-images.githubusercontent.com/105562242/169313712-d56cd8c3-d132-49e6-8630-ae26cde773bf.png">

<img width="1720" alt="Screenshot 2022-05-19 at 3 06 59 PM" src="https://user-images.githubusercontent.com/105562242/169313845-5eb6962b-4d0a-44bd-9a9a-7aebc478b890.png">

<img width="1725" alt="Screenshot 2022-05-19 at 7 39 52 PM" src="https://user-images.githubusercontent.com/105562242/169314129-0e6cd2ce-6753-474f-9e9c-bc568c1b5818.png">






 

 
 

 


