
By now We have learned what Load Balancing is used for and have configured an LB solution using Apache, but a DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. That is why, in this project we will configure an Nginx Load Balancer solution.

It is also extremely important to ensure that connections to our Web solutions are secure and information is encrypted in transit – we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.

When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.

This threat is real – users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

There are different types of SSL/TLS certificates – you can learn more about them here. You can also watch a tutorial on how SSL works here or an additional resource here

In this project you will register your website with LetsEnrcypt Certificate Authority, to automate certificate issuance you will use a shell client recommended by LetsEncrypt – cetrbot.

Task
This project consists of two parts:

Configure Nginx as a Load Balancer
Register a new domain name and configure secured connection using SSL/TLS certificates
Our target architecture will look like this:

<img width="1126" alt="Screenshot 2022-07-16 at 3 34 03 PM" src="https://user-images.githubusercontent.com/105562242/179350285-26e414e5-977a-49f7-ac3b-e65123d5a80b.png">

Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```
Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```

Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

In order to get a valid SSL certificate – we need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.

Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

Assign an Elastic IP to your Nginx LB server and associate our domain name with this Elastic IP

Update A record in registrar to point to Nginx LB using Elastic IP address

Check that our Web Servers can be reached from our browser using new domain name using HTTP protocol – http://<your-domain-name.com>

Configure Nginx to recognize our new domain name

Update our nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

<img width="890" alt="Screenshot 2022-07-15 at 7 12 34 PM" src="https://user-images.githubusercontent.com/105562242/179350515-03f0dff1-0311-4805-9648-0e07c67d0e72.png">

<img width="895" alt="Screenshot 2022-07-15 at 7 13 02 PM" src="https://user-images.githubusercontent.com/105562242/179350524-96b90e18-42c7-4eb7-8fc7-75fb6bbf276a.png">

<img width="681" alt="Screenshot 2022-07-15 at 7 16 34 PM" src="https://user-images.githubusercontent.com/105562242/179350566-82532364-5a67-4fa6-afc1-2cca91554e60.png">

<img width="631" alt="Screenshot 2022-07-15 at 7 16 45 PM" src="https://user-images.githubusercontent.com/105562242/179350582-7a499e28-3447-4a2b-812a-c26f50712035.png">

<img width="1157" alt="Screenshot 2022-07-15 at 7 36 48 PM" src="https://user-images.githubusercontent.com/105562242/179350602-1be81d07-672f-43a6-8c5b-25a9e5f4ff54.png">

<img width="1616" alt="Screenshot 2022-07-15 at 7 37 40 PM" src="https://user-images.githubusercontent.com/105562242/179350610-32c6002d-fe8d-4d51-bc70-566c09a855e8.png">

<img width="761" alt="Screenshot 2022-07-15 at 7 41 28 PM" src="https://user-images.githubusercontent.com/105562242/179350624-701216a0-ae47-4202-85bd-465f1328b1fe.png">

Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running
```
sudo systemctl status snapd
```
Install certbot
```
sudo snap install --classic certbot
```
Request our certificate (just follow the certbot instructions – we will need to choose which domain we want our certificate to be issued for, domain name will be looked up from nginx.conf

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
Test secured access to our Web Solution by trying to reach https://<your-domain-name.com>

We shall be able to access our website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in our browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for our website.

<img width="1126" alt="Screenshot 2022-07-15 at 7 45 59 PM" src="https://user-images.githubusercontent.com/105562242/179351225-7fff8266-ad3e-495b-9f91-7181cb146775.png">

<img width="1456" alt="Screenshot 2022-07-15 at 7 48 26 PM" src="https://user-images.githubusercontent.com/105562242/179351230-d80ae19c-a2b5-43b7-90c8-09f95308214b.png">

Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

We can test renewal command in dry-run mode
```
sudo certbot renew --dry-run
```
Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

```
crontab -e
```
Add following line:
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

<img width="888" alt="Screenshot 2022-07-15 at 7 53 42 PM" src="https://user-images.githubusercontent.com/105562242/179351591-17127281-3ce0-4198-89d9-dc27f0c464fa.png">

