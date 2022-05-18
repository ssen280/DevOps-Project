## LEMP (Linux, NGINX, MySQL, PHP) stack implementation in AWS.



## We will follow below steps :


**1. We will create linux EC2 instance in AWS**

**2. We will install NGINX, MySQL and PHP in linux (Ubuntu) OS**

**3. We will open port 80 to acceess web page from outside using EC2 instance public IP address.** 

----------------------

### Creating EC2 istance in AWS & Installing the Nginx Web Server :

##### In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package.

##### Since this is our first time using apt for this session, start off by updating our server’s package index. Following that, we can use apt install to get Nginx installed.

##### We will open port 80 in security group as inboud bound rule so that we can access web page from web.

```
sudo apt update
sudo apt install nginx
```
##### To check if NGINX running :
```
sudo systemctl status nginx
```
<img width="1433" alt="Screenshot 2022-05-15 at 7 14 30 PM" src="https://user-images.githubusercontent.com/105562242/168476055-1958e967-df76-4719-909f-199a76f90ba8.png">

<img width="1410" alt="Screenshot 2022-05-15 at 7 13 06 PM" src="https://user-images.githubusercontent.com/105562242/168476001-f0ba00fc-9882-4da2-b933-a3258ead37d7.png">

<img width="883" alt="Screenshot 2022-05-15 at 7 20 06 PM" src="https://user-images.githubusercontent.com/105562242/168476303-cc5c9858-7d9e-4c21-933a-b38a438a78d8.png">

<img width="891" alt="Screenshot 2022-05-15 at 7 21 16 PM" src="https://user-images.githubusercontent.com/105562242/168476367-f3675b8c-4133-43be-be4e-b62658ef7565.png">

##### Our server is running and we can access it locally and from the Internet (Source 0.0.0.0/0 means ‘from any IP address’).

##### First, let us try to check how we can access it locally in our Ubuntu shell, run:

<img width="885" alt="Screenshot 2022-05-15 at 7 39 24 PM" src="https://user-images.githubusercontent.com/105562242/168477315-86be23f9-5789-4e3b-8ed6-66ba703f447b.png">

##### Now it is time for us to test how our Nginx server can respond to requests from the Internet.
##### Open a web browser of your choice and try to access following url

<img width="1386" alt="Screenshot 2022-05-15 at 7 48 09 PM" src="https://user-images.githubusercontent.com/105562242/168477632-1299c174-c8df-46bf-be44-b9f8a6f62471.png">

### Installing MYSQL:

##### Now that we have a web server up and running, we need to install a Database Management System (DBMS) to be able to store and manage data for our site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.

```
sudo apt install mysql-server
sudo mysql

```

<img width="882" alt="Screenshot 2022-05-15 at 8 01 46 PM" src="https://user-images.githubusercontent.com/105562242/168478168-2b297430-f30a-4712-867f-ad20c64a96ab.png">

##### It’s recommended that we run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to our database system. Before running the script we will set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as PassWord.1.

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

<img width="885" alt="Screenshot 2022-05-15 at 8 10 10 PM" src="https://user-images.githubusercontent.com/105562242/168478950-bcc010f1-e8ef-46bc-b8f7-497d5f847c90.png">

### Installing PHP

##### You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

##### While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. We’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, we’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

##### To install these 2 packages at once, run:
```
sudo apt install php-fpm php-mysql
```
### Configuring Nginx to Use PHP Processor

##### When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use web-lemp as an example domain name.

##### Create the root web directory for web-lemp as follows:

```
sudo mkdir /var/www/web-lemp
```

##### Next, assign ownership of the directory with the $USER environment variable, which will reference current system user:(we are using root user hence not required)

```
sudo chown -R $USER:$USER /var/www/web-lemp
```

##### Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor.

```
sudo vim /etc/nginx/sites-available/web-lemp
```

##### This will create a new blank file. Paste in the following bare-bones configuration:

<img width="888" alt="Screenshot 2022-05-16 at 1 07 38 AM" src="https://user-images.githubusercontent.com/105562242/168490937-b5bcc96d-5b55-4b9a-8d55-fd453539dce7.png">

```
sudo ln -s /etc/nginx/sites-available/web-lemp /etc/nginx/sites-enabled/
sudo nginx -t
```
##### We shall see following message:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```
##### Activate  configuration by linking to the config file from Nginx’s sites-enabled directory:

<img width="889" alt="Screenshot 2022-05-15 at 11 23 57 PM" src="https://user-images.githubusercontent.com/105562242/168486925-f7da77f6-8ceb-4403-a01e-df2ab2de72b2.png">

<img width="884" alt="Screenshot 2022-05-15 at 11 24 37 PM" src="https://user-images.githubusercontent.com/105562242/168486944-c7acf4f8-6b43-4fd1-bb3c-e80f0a6a44bb.png">

##### We will create index.html file to /var/www/web-lemp


<img width="873" alt="Screenshot 2022-05-15 at 11 43 17 PM" src="https://user-images.githubusercontent.com/105562242/168487705-70268d6e-322f-4928-b703-d77f1931a095.png">


##### We are able to access page now 

<img width="1059" alt="Screenshot 2022-05-15 at 11 45 07 PM" src="https://user-images.githubusercontent.com/105562242/168487731-0c8cc242-2d62-4e41-911c-3f88da865f11.png">


##### We can leave this file in place as a temporary landing page for our application until we set up an index.php file to replace it. Once we do that, remember to remove or rename the index.html file, as it would take precedence over an index.php file by default.


### Testing PHP with Nginx

##### We can test it to validate that Nginx can correctly hand .php files off to our PHP processor.
##### We can do this by creating a test PHP file in our document root. Open a new file called info.php within our document root in text editor:

```
sudo nano /var/www/web-lemp/index.php
```
```
Type or paste the following lines into the new file. This is valid PHP code that will return information about server:

<?php
phpinfo();

```

<img width="886" alt="Screenshot 2022-05-16 at 1 06 13 AM" src="https://user-images.githubusercontent.com/105562242/168490789-58f807ac-33ac-462d-aa87-00f0c2970c8f.png">


<img width="1286" alt="Screenshot 2022-05-16 at 1 14 54 AM" src="https://user-images.githubusercontent.com/105562242/168491127-f1ff781c-bebf-4fc9-98d7-f97cbb44149b.png">

##### Retrieving data from MySQL database with PHP:

##### In this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

##### At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

##### We will create a database named example_database and a user named example_user, but you can replace these names with different values

##### First, connect to the MySQL console using the root account:
```
sudo mysql
```
##### To create a new database, run the following command from your MySQL console:
```
mysql> CREATE DATABASE `example_database`;
```
##### Now you can create a new user and grant him full privileges on the database you have just created.

##### The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.
```
mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```
##### Now we need to give this user permission over the example_database database:
```
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
```
##### This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.

##### Now exit the MySQL shell with:
```
mysql> exit
```
##### We can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:
```
mysql -u example_user -p
```
##### Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database:
```
mysql> SHOW DATABASES;
```

<img width="890" alt="Screenshot 2022-05-16 at 1 28 11 AM" src="https://user-images.githubusercontent.com/105562242/168492113-1f3ddcaf-1f60-4715-bac3-4b12b8f05bc9.png">

<img width="885" alt="Screenshot 2022-05-16 at 1 29 20 AM" src="https://user-images.githubusercontent.com/105562242/168492156-c8aee3c2-b715-4a1b-a77d-7688b7ff6da0.png">


##### Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:
```
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
##### Insert a few rows of content in the test table. We might want to repeat the next command a few times, using different VALUES:
```
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```
##### To confirm that the data was successfully saved to table, run:
```
mysql>  SELECT * FROM example_database.todo_list;
```

<img width="888" alt="Screenshot 2022-05-16 at 1 34 08 AM" src="https://user-images.githubusercontent.com/105562242/168492255-e66e157e-9c92-4d14-bb38-9f1b55c0685b.png">

##### Now we can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in  custom web root directory using preferred editor. We’ll use vi for that:
```
nano /var/www/projectLEMP/todo_list.php
```
##### The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

<img width="889" alt="Screenshot 2022-05-16 at 1 37 01 AM" src="https://user-images.githubusercontent.com/105562242/168492315-7065624b-f55a-4cd1-a646-ee8c1403db64.png">

##### We should see a page like this, showing the content we’ve inserted in  test table:


<img width="973" alt="Screenshot 2022-05-16 at 1 51 30 AM" src="https://user-images.githubusercontent.com/105562242/168492351-e27c1362-a0c3-4f84-9491-4ab96b520c97.png">










