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



