
### Connecting ec2 instace from VS Code. Installing Nginx with package manager, Building nginx from source & adding modules 
---------------------------------------------

<img width="871" alt="Screenshot 2023-03-06 at 8 23 31 AM" src="https://user-images.githubusercontent.com/105562242/223008772-64c5c175-afb8-4550-826f-d82fda95ef54.png">


<img width="1061" alt="Screenshot 2023-03-06 at 5 51 50 AM" src="https://user-images.githubusercontent.com/105562242/222994699-90c00899-d84b-4672-8de3-853767ec56f6.png">

<img width="1088" alt="Screenshot 2023-03-06 at 5 52 51 AM" src="https://user-images.githubusercontent.com/105562242/222994755-88466785-ddc5-40d9-8fc3-7a484aea9abd.png">

<img width="1243" alt="Screenshot 2023-03-06 at 5 54 58 AM" src="https://user-images.githubusercontent.com/105562242/222994880-ea43ea1c-5bba-4786-97a1-c91a96143688.png">

<img width="1179" alt="Screenshot 2023-03-06 at 5 55 10 AM" src="https://user-images.githubusercontent.com/105562242/222994897-99a1aa37-b16b-4f24-8ee8-0f2d2c853504.png">

<img width="1249" alt="Screenshot 2023-03-06 at 6 00 31 AM" src="https://user-images.githubusercontent.com/105562242/222995235-32c438a9-9acc-4db4-97ff-8f628a669d0d.png">

<img width="1082" alt="Screenshot 2023-03-06 at 6 01 50 AM" src="https://user-images.githubusercontent.com/105562242/222995308-eb632944-b369-42ea-b40f-8371bae09b0f.png">

```tar -zxvf nginx-1.23.3.tar.gz ``` 

### We have to run ./configure script to check all necessay tools are installed.To install compiler and all necessary tools : ```tar -zxvf nginx-1.23.3.tar.gz``` need to run. Post we will run ./configure script again to check all are good. 



<img width="740" alt="Screenshot 2023-03-06 at 6 07 54 AM" src="https://user-images.githubusercontent.com/105562242/222995659-44bf75b6-4de3-4fd3-a1fc-e31bf7fc8485.png">

<img width="1087" alt="Screenshot 2023-03-06 at 6 06 19 AM" src="https://user-images.githubusercontent.com/105562242/222995559-56dc6017-4d12-419d-8dce-4f745722fe1a.png">



### Below libary is missing. we will install it. ```apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev``` We will run again ./configure script and we will see all good.



<img width="860" alt="Screenshot 2023-03-06 at 6 10 48 AM" src="https://user-images.githubusercontent.com/105562242/222995862-60342105-9f63-49a7-94d3-654100184e0a.png">

<img width="1089" alt="Screenshot 2023-03-06 at 6 14 12 AM" src="https://user-images.githubusercontent.com/105562242/222996123-5ac223aa-4f67-492a-9f7f-e2762cbd163f.png">



### Please refer :- https://nginx.org/en/docs/configure.html. we will configure paths. 



``` ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module```

<img width="1091" alt="Screenshot 2023-03-06 at 6 23 39 AM" src="https://user-images.githubusercontent.com/105562242/222996775-783ec8d4-1c0d-40b3-aba9-ce6d1c0602fc.png">

<img width="1018" alt="Screenshot 2023-03-06 at 6 23 59 AM" src="https://user-images.githubusercontent.com/105562242/222996800-e71e1d15-a829-4e41-bf92-6fffa3470ed9.png">


### Now we will ```make``` and also run ```make install```



<img width="1083" alt="Screenshot 2023-03-06 at 6 27 04 AM" src="https://user-images.githubusercontent.com/105562242/222997007-5465958e-a38d-4f71-8052-dff1a7ac28e3.png">

<img width="1088" alt="Screenshot 2023-03-06 at 6 28 20 AM" src="https://user-images.githubusercontent.com/105562242/222997111-6ad30e38-59ac-44c6-a7cf-3842ff751cc6.png">



### To verify we will check version and configuratin files under /etc/nginx. We will check nginx running or not post that.



<img width="942" alt="Screenshot 2023-03-06 at 6 30 12 AM" src="https://user-images.githubusercontent.com/105562242/222997271-93096442-8c14-42e2-a6d7-5439b28261a2.png">

<img width="1087" alt="Screenshot 2023-03-06 at 6 31 24 AM" src="https://user-images.githubusercontent.com/105562242/222997342-4cc9959e-b931-48d2-912c-4fdf5b30d504.png">



### Adding nginx service :

### To stop nginx service :

<img width="1087" alt="Screenshot 2023-03-06 at 6 39 23 AM" src="https://user-images.githubusercontent.com/105562242/222997859-836271a7-a18c-4c8e-a996-d96f557e716a.png">

<img width="1086" alt="Screenshot 2023-03-06 at 6 39 40 AM" src="https://user-images.githubusercontent.com/105562242/222997893-a9e80846-2d59-4daa-aee1-b441484387a4.png">



### We will add systemd service ( First we need to check if we are able to control nginx service with systemctl command. if we are able to control systemctl command then we can skip this part. we have to excute below steps in-case we are not able to control nginx service with systemctl command). To enable nginx service during boot. We have to enable with systemctl command ```systemctl enable nginx```
--------------------------------------------------------------------


https://www.freedesktop.org/wiki/Software/systemd/

https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/

https://www.nginx.com/resources/wiki/start/topics/examples/systemd/

### We will create and go to path /lib/systemd/system/nginx.service and do changes services paths as below

<img width="846" alt="Screenshot 2023-03-06 at 6 48 20 AM" src="https://user-images.githubusercontent.com/105562242/222998671-672ebe79-f018-435a-b4b4-ccf249fdcfcf.png">


<img width="1087" alt="Screenshot 2023-03-06 at 6 46 29 AM" src="https://user-images.githubusercontent.com/105562242/222998495-fe0de508-4d14-4d83-b730-6c7c84de1e8c.png">

<img width="1072" alt="Screenshot 2023-03-06 at 7 02 30 AM" src="https://user-images.githubusercontent.com/105562242/222999978-6d054169-9c81-47cc-9b2a-93ee56273758.png">



### Configuring vitual host : We will do configuration change in /etc/nginx/nginx.conf file and direct the path to our index file. Then we will reload nginx. please note : stop start needs downtime but reload does not hence we will use ```systemctl reload nginx```

```
events {}

http {

  include mime.types;

  server {

    listen 80;
    server_name 34.234.201.126;

    root /sites/demo;
  }
}

```

### To check configuration is ok post changes. we will run ```nginx -t```

<img width="866" alt="Screenshot 2023-03-06 at 8 55 09 AM" src="https://user-images.githubusercontent.com/105562242/223012638-ad1d52dd-2515-4c5c-9b0c-741d41deb1bb.png">

<img width="798" alt="Screenshot 2023-03-06 at 8 54 30 AM" src="https://user-images.githubusercontent.com/105562242/223012557-19ddc601-6b78-4019-80c9-9175d8939c51.png">

### To verify if sytle.css file loaded perfeclty not we have to curl our web address. Here we can see Content-Type: text/css

<img width="740" alt="Screenshot 2023-03-06 at 8 57 18 AM" src="https://user-images.githubusercontent.com/105562242/223012905-af2745c5-33d7-4a8b-b855-eb888e482ca4.png">

### To fix any type we have to use mime.types. below we can see it includes all file types hence we have mentioned the same in nginx.conf file ```include mime.types;```

<img width="713" alt="Screenshot 2023-03-06 at 8 59 38 AM" src="https://user-images.githubusercontent.com/105562242/223013182-8ac6fb25-5f90-4691-81dc-80c0effabb45.png">



### Location blocks : this is require when we navigate to another file. means static web page to another page. below we can see from static web page when we are navigating to another page its giving error which we will fix it. 


<img width="1324" alt="Screenshot 2023-03-06 at 9 05 11 AM" src="https://user-images.githubusercontent.com/105562242/223013849-eaf7afc0-a386-42ed-b5b9-5c5eaf2a6312.png">

<img width="1076" alt="Screenshot 2023-03-06 at 9 04 13 AM" src="https://user-images.githubusercontent.com/105562242/223013721-7aa523ee-7d35-401a-a394-9e9c2edcdd64.png">

### Please refer below sintex for prefix match to navigate to another page

```
   # Preferential Prefix match
    location ^~ /Greet2 {
      return 200 'Hello from NGINX "/greet" location.';
    }

    # # Exact match
    # location = /greet {
    #   return 200 'Hello from NGINX "/greet" location - EXACT MATCH.';
    # }

    # # REGEX match - case sensitive
    # location ~ /greet[0-9] {
    #   return 200 'Hello from NGINX "/greet" location - REGEX MATCH.';
    # }

    # REGEX match - case insensitive
    location ~* /greet[0-9] {
      return 200 'Hello from NGINX "/greet" location - REGEX MATCH INSENSITIVE.';
      
 ```

<img width="906" alt="Screenshot 2023-03-06 at 9 34 46 AM" src="https://user-images.githubusercontent.com/105562242/223017097-391477e9-c3a5-44b1-a1df-7fa9b43ef8d9.png">

<img width="793" alt="Screenshot 2023-03-06 at 9 35 01 AM" src="https://user-images.githubusercontent.com/105562242/223017128-443e4eaa-ccb4-4bd5-ba35-c76ffb95f420.png">

<img width="863" alt="Screenshot 2023-03-06 at 9 35 30 AM" src="https://user-images.githubusercontent.com/105562242/223017168-529eb84b-c94b-4878-a797-bd273a05d48f.png">


### Variables : There are two type of variables 1. We create our own variables 2. nginx in-build variables 

### Nginx build in variables : https://nginx.org/en/docs/varindex.html

### Here we can see below variables are in-build : $host , $uri, $args

<img width="669" alt="Screenshot 2023-03-07 at 7 42 13 AM" src="https://user-images.githubusercontent.com/105562242/223301645-04cfa647-b0e0-4787-8712-552958aa6402.png">

<img width="768" alt="Screenshot 2023-03-07 at 7 42 26 AM" src="https://user-images.githubusercontent.com/105562242/223301681-5e77c42a-d2bf-4690-8437-c94a67cbc1a0.png">

### Here we can use our own variable and declare it. 

<img width="701" alt="Screenshot 2023-03-07 at 8 03 09 AM" src="https://user-images.githubusercontent.com/105562242/223304666-022c39e3-c800-487c-81f1-d1537ef4452c.png">

<img width="644" alt="Screenshot 2023-03-07 at 8 03 25 AM" src="https://user-images.githubusercontent.com/105562242/223304702-92f64115-be2b-4ba1-814a-738f27b4306f.png">

### Few more exampls : https://github.com/ssen280/NGINX-CONFIGURATION-CONCEPT/blob/main/cnfiguration/03%2BVariables.conf

### Rewrites & Redirects : return statement take a status code (200) and return a string but but with return statement takes a redirect code (307) then it return a path

<img width="787" alt="Screenshot 2023-03-07 at 8 13 36 AM" src="https://user-images.githubusercontent.com/105562242/223306724-97645a8a-0a18-4c89-9e35-3a5bcae50077.png">

<img width="776" alt="Screenshot 2023-03-07 at 8 15 14 AM" src="https://user-images.githubusercontent.com/105562242/223306951-49998987-11bc-41d6-9668-696f5fa37f09.png">

<img width="771" alt="Screenshot 2023-03-07 at 8 16 25 AM" src="https://user-images.githubusercontent.com/105562242/223307126-7d220d96-2321-4e3e-9e2b-a58ab11a949b.png">

<img width="462" alt="Screenshot 2023-03-07 at 8 19 01 AM" src="https://user-images.githubusercontent.com/105562242/223307512-bf3ecff3-478c-4fae-882a-c0eb4c2c1cac.png">

<img width="1005" alt="Screenshot 2023-03-07 at 8 19 16 AM" src="https://user-images.githubusercontent.com/105562242/223307544-fd3625f5-e9f7-4d0e-b8ec-9b37702e8320.png">

### try_files : it tries aurument one by one. and which is true display that. when we use $arg it checks absolute path and go each argument one by one and only last argument do the re-write. /nothing path does not exist hence last argument did the re-write


<img width="791" alt="Screenshot 2023-03-07 at 8 47 11 AM" src="https://user-images.githubusercontent.com/105562242/223311597-1719eaec-7aad-4481-925a-a4d1d8cb1e3b.png">

<img width="751" alt="Screenshot 2023-03-07 at 8 47 22 AM" src="https://user-images.githubusercontent.com/105562242/223311629-b692232c-8c47-4967-9842-ba8ef44c5f59.png">

### Logging : Nginx provides two logs type : 1. error.log 2.access.log. log path : /var/log/nginx

<img width="771" alt="Screenshot 2023-03-07 at 8 53 54 AM" src="https://user-images.githubusercontent.com/105562242/223312556-7be498b0-14dd-486a-a2f1-fe5e9bee0db4.png">


<img width="832" alt="Screenshot 2023-03-07 at 8 59 09 AM" src="https://user-images.githubusercontent.com/105562242/223313336-8f730b57-ca48-42c5-a396-2f14c3a55517.png">



<img width="686" alt="Screenshot 2023-03-07 at 9 02 22 AM" src="https://user-images.githubusercontent.com/105562242/223313780-4fdb89c1-4dcb-47f3-9960-45a903a84fa5.png">


<img width="973" alt="Screenshot 2023-03-07 at 9 04 11 AM" src="https://user-images.githubusercontent.com/105562242/223314029-5113fe74-7f85-4ed7-8301-777e35a7b68c.png">

<img width="723" alt="Screenshot 2023-03-07 at 9 04 32 AM" src="https://user-images.githubusercontent.com/105562242/223314075-d6582eea-8e48-471e-be24-417a6dd93404.png">


### Inheritince & Directive Types : 1. Standard 2. Array 3. Action . Please refer github source code for code details 

### PHP Processing : Here we see how to configure dynamic site content where we use php. here requiest goes to nginx first then nginx forward the request to php content and php concent then send the dynamic content to nginx and nginx shows the dynamic content to requester. 

#### First we will install php on server 

<img width="970" alt="Screenshot 2023-03-07 at 8 03 11 PM" src="https://user-images.githubusercontent.com/105562242/223452942-7bf8d253-f8ba-4d46-a335-8c6d16f71f43.png">

<img width="977" alt="Screenshot 2023-03-07 at 8 04 32 PM" src="https://user-images.githubusercontent.com/105562242/223453298-ae1dc622-9d1f-418b-82c9-9286b7335695.png">

<img width="974" alt="Screenshot 2023-03-07 at 8 05 15 PM" src="https://user-images.githubusercontent.com/105562242/223453508-c6e9ef14-08de-4847-befe-47cfdb9ccc4a.png">

<img width="739" alt="Screenshot 2023-03-07 at 8 28 11 PM" src="https://user-images.githubusercontent.com/105562242/223459875-a11d853e-b5e1-44e0-b25a-d785e76b0af5.png">

<img width="893" alt="Screenshot 2023-03-07 at 8 30 28 PM" src="https://user-images.githubusercontent.com/105562242/223460456-b4f62fc6-cb28-4d65-b6fa-4fc5d8f53c8d.png">

<img width="1086" alt="Screenshot 2023-03-07 at 8 30 43 PM" src="https://user-images.githubusercontent.com/105562242/223460515-8b899a33-a8f2-4a23-88b4-a969ea82ee09.png">

<img width="975" alt="Screenshot 2023-03-07 at 8 33 23 PM" src="https://user-images.githubusercontent.com/105562242/223461256-4402171c-7c4b-4e3a-a41e-2315bbd519b3.png">


#### Here we can see nginx and php processs are running under same user hence there is not issue to access php info file. incase we get error while accessig info.php file then we have to check if both processes are running same user and we have to correct it. 

#### Here we have added the user name in nginx.conf file instead of using chmod command to change user permission

<img width="717" alt="Screenshot 2023-03-07 at 8 39 01 PM" src="https://user-images.githubusercontent.com/105562242/223462859-10bcc8c7-d73b-4f5e-86f2-cdb6a8302ac8.png">


<img width="974" alt="Screenshot 2023-03-07 at 8 34 37 PM" src="https://user-images.githubusercontent.com/105562242/223461613-c9d027fe-6403-4e6e-a722-78d331b67587.png">

<img width="1187" alt="Screenshot 2023-03-07 at 8 36 32 PM" src="https://user-images.githubusercontent.com/105562242/223462169-f6238468-2bf3-44ab-aa36-315c54450527.png">

#### Worker Processes : There are two processes 1. Master processes 2. worker processes. if we want we can increase workers processes. We can increase worer processes by mentoning it in nginx.conf file and we can set to auto as well 

<img width="965" alt="Screenshot 2023-03-07 at 9 02 49 PM" src="https://user-images.githubusercontent.com/105562242/223469626-f0a5ae9e-0cbe-492d-b1d2-00982ac465df.png">

<img width="509" alt="Screenshot 2023-03-07 at 9 06 29 PM" src="https://user-images.githubusercontent.com/105562242/223470658-711b03a3-299c-4a23-9c8e-31c02c677b29.png">

<img width="575" alt="Screenshot 2023-03-07 at 9 10 18 PM" src="https://user-images.githubusercontent.com/105562242/223471869-518b23bd-a3f8-4c22-97db-f4f51676ba55.png">

<img width="997" alt="Screenshot 2023-03-07 at 9 11 03 PM" src="https://user-images.githubusercontent.com/105562242/223472089-a47b4bf0-98cf-4a17-83a4-65d189ade69d.png">
