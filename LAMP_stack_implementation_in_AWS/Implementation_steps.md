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

##### In this project, we will set up a domain called web-lamp


















