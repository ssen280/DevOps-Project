#### In project-7 We have created three web server to acess tooling web page. To access the web page we have to use individual web servers in this case when one web server will be down then it will be troublesome, instead of doging this we will add loadbalancer server which will help to use single web site link with loadbalacer server ip address and we will never face web page down issue when one of web servers will be down.


#### In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.


<img width="1054" alt="Screenshot 2022-07-15 at 12 19 31 AM" src="https://user-images.githubusercontent.com/105562242/179060211-ff1b734b-1547-4d3e-9b11-c145bee64314.png">

<img width="1105" alt="Screenshot 2022-07-15 at 12 20 24 AM" src="https://user-images.githubusercontent.com/105562242/179060358-21388786-f623-4563-b44d-2871b7ade266.png">

##### Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb.
##### Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.
 <img width="1211" alt="Screenshot 2022-07-15 at 12 26 37 AM" src="https://user-images.githubusercontent.com/105562242/179061486-b3ba497c-ae36-4661-9916-5a3ef45509d3.png">

##### Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```
##### Make sure apache2 is up and running

`sudo systemctl status apache2`

##### Configure load balancing

```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server

sudo systemctl restart apache2

```
