## LAMP (Linux, Apache, MySQL, PHP) stack implementation in AWS.



## We will follow below steps :


**1. We will create linux EC2 instance in AWS**

**2. We will install Apache, MySQL and PHP in linux (Ubuntu) OS**

**3. We will open port 80 to acceess web page from outside using EC2 instance public IP address.** 

----------------------

### Creating linux EC2 instances in AWS :

1. We will create Ubuntu linux EC2 instance and give name tag as web-LAMP

<img width="1728" alt="ec2" src="https://user-images.githubusercontent.com/105562242/168448670-2ccabb58-7c3c-4f38-8fdb-c1e281811901.png">

2. We will open port 80 to access web page from outside, using EC2 instance's public ip address

<img width="1393" alt="Screenshot 2022-05-15 at 3 04 03 AM" src="https://user-images.githubusercontent.com/105562242/168448812-b75e5a36-1245-4fec-bab0-85990bde8037.png">

### Installing Apache in OS

##### Apache HTTP Server is the most widely used web server software. Developed and maintained by Apache Software Foundation, Apache is an open source software available for free. It runs on 67% of all webservers in the world. It is fast, reliable, and secure.

* update a list of packages in package manager
  ##### sudo apt update

* run apache2 package installation
  ##### sudo apt install apache2
* Check the status 
  ##### sudo systemctl status apache2
  
  <img width="888" alt="Screenshot 2022-05-15 at 2 04 34 PM" src="https://user-images.githubusercontent.com/105562242/168464216-aad6d4ab-b044-4d29-bd77-bb1cabbb267b.png">
  
* We can access page from internet browser with public IP address of EC2 instace on port 80 

<img width="1257" alt="Screenshot 2022-05-15 at 2 06 14 PM" src="https://user-images.githubusercontent.com/105562242/168464316-f2ef6d78-b02b-464b-89e0-33263fb03d1a.png">

### Installing MySQL in OS
##### Now that we have a web server up and running, we need to install a Database Management System (DBMS) to be able to store and manage data for our site in a relational database. MySQL is a popular relational database management system used within PHP environments

##### sudo apt install mysql-server
##### sudo mysql
<img width="886" alt="Screenshot 2022-05-15 at 2 13 35 PM" src="https://user-images.githubusercontent.com/105562242/168464556-21128dcc-c6ea-49e8-ada1-0420c852cd4b.png">

* We will set password for root user 

##### ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

<img width="885" alt="Screenshot 2022-05-15 at 2 18 13 PM" src="https://user-images.githubusercontent.com/105562242/168464741-8262047c-fbb6-423b-8334-9dd0ab60728f.png">

### Installing PHP in OS

##### we have Apache installed to serve our content and MySQL installed to store and manage our data. PHP is the component of our setup that will process code to display dynamic content to the end user. In addition to the php package, we’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. We’ll also need libapache2-mod-php to enable Apache to handle PHP files. Core PHP packages will automatically be installed as dependencies.

##### apt install php libapache2-mod-php php-mysql
##### php --version
<img width="885" alt="Screenshot 2022-05-15 at 2 25 10 PM" src="https://user-images.githubusercontent.com/105562242/168464958-8bdd577b-4ce9-4006-8070-0368eb429090.png">

##### To test our setup with a PHP script, it’s best to set up a proper Apache Virtual Host to hold our website’s files and folders. Virtual host allows us to have multiple websites located on a single machine and users of the websites will not even notice it.

<img width="1046" alt="Screenshot 2022-05-15 at 2 27 53 PM" src="https://user-images.githubusercontent.com/105562242/168465023-117d3098-ac7a-4b24-9ddd-ca5bcdfcb993.png">

### Creating a virtual host for our website using apache

#### In this project, we will set up a domain called web-lamp

##### Create the directory for projectlamp using ‘mkdir’ command and provide owner ship as follows(Here we are using root user hence ownership change is not required):

##### sudo mkdir /var/www/projectlamp
#####  sudo chown -R $USER:$USER /var/www/projectlamp. ( prefered username & group name)

<img width="848" alt="Screenshot 2022-05-15 at 2 33 16 PM" src="https://user-images.githubusercontent.com/105562242/168465351-5f1ed522-e0a0-4090-8d23-f65efa90a527.png">

##### Then, create and open a new configuration file in Apache’s sites-available directory using preferred command-line editor

##### sudo vi /etc/apache2/sites-available/web-lamp.conf

<img width="887" alt="Screenshot 2022-05-15 at 2 40 45 PM" src="https://user-images.githubusercontent.com/105562242/168465586-3d70d60e-9be3-4ba0-9750-35a470f8c775.png">


##### now use a2ensite command to enable the new virtual host:

##### sudo a2ensite web-lamp

##### We might want to disable the default website that comes installed with Apache. This is required if we are not using a custom domain name, because in this case Apache’s default configuration would overwrite our virtual host. To disable Apache’s default website use a2dissite command

##### sudo a2dissite 000-default

##### To make sure our configuration file doesn’t contain syntax errors, run:

##### sudo apache2ctl configtest

##### Finally, reload Apache so these changes take effect:

##### sudo systemctl reload apache2


<img width="807" alt="Screenshot 2022-05-15 at 2 44 31 PM" src="https://user-images.githubusercontent.com/105562242/168465814-ba529d83-c315-47fb-ad12-88ac5f6eb089.png">


##### but the web root /var/www/projectlamp is still empty. We will create an index.html file in that location so that we can test that the virtual host works as expected:

<img width="886" alt="Screenshot 2022-05-15 at 2 53 34 PM" src="https://user-images.githubusercontent.com/105562242/168466037-a6c3f2d1-450d-423d-815e-8e84527cdd17.png">
<img width="896" alt="Screenshot 2022-05-15 at 2 54 47 PM" src="https://user-images.githubusercontent.com/105562242/168466039-d24862fd-ea18-421d-8506-8c9f28ae5c3b.png">

<img width="992" alt="Screenshot 2022-05-15 at 2 55 10 PM" src="https://user-images.githubusercontent.com/105562242/168466046-ca6dfdba-5a50-416f-ab7d-baccc82f4502.png">

<img width="862" alt="Screenshot 2022-05-15 at 2 58 10 PM" src="https://user-images.githubusercontent.com/105562242/168466152-2abcfbbd-d471-4cd0-9fc4-933b7771a0d1.png">
<img width="1064" alt="Screenshot 2022-05-15 at 2 58 22 PM" src="https://user-images.githubusercontent.com/105562242/168466156-0189a4c4-8bb6-47cc-8de1-e723d0873e2d.png">


##### We can leave this file in place as a temporary landing page for your application until we set up an index.php file to replace it. Once we do that, remember to remove or rename the index.html file from our document root, as it would take precedence over an index.php file by default.

#### Enable PHP on the website

##### With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file. This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors. Because this page will take precedence over the index.php page, it will then become the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root, bringing back the regular application page.

##### In case we want to change this behavior, we’ll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:

##### sudo vim /etc/apache2/mods-enabled/dir.conf

```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
##### After saving and closing the file, we will need to reload Apache so the changes take effect:

##### sudo systemctl reload apache2

##### Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server.

##### Now that we have a custom location to host our website’s files and folders, we’ll create a PHP test script to confirm that Apache is able to handle and process requests for PHP files.

##### Create a new file named index.php inside custom web root folder:

``` vim /var/www/projectlamp/index.php ```
##### This will open a blank file. Add the following text, which is valid PHP code, inside the file:

``` 
<?php
phpinfo(); 
```

<img width="888" alt="Screenshot 2022-05-15 at 3 07 04 PM" src="https://user-images.githubusercontent.com/105562242/168466723-9850ba44-6694-42b3-8eff-e8b07a4a4eca.png">


<img width="1108" alt="Screenshot 2022-05-15 at 3 07 20 PM" src="https://user-images.githubusercontent.com/105562242/168466731-3713225c-14cf-428c-9079-a81bb5bc65ce.png">


##### This page provides information about our server from the perspective of PHP. It is useful for debugging and to ensure that your settings are being applied correctly.

##### If we can see this page in browser, then  PHP installation is working as expected.

##### After checking the relevant information about PHP server through that page, it’s best to remove the file we created as it contains sensitive information about our PHP environment -and our Ubuntu server. we can use rm to do so:

``` sudo rm /var/www/projectlamp/index.php ```




















