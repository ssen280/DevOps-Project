
#### Connecting ec2 instace from VS Code. Installing Nginx with package manager, Building nginx from source & adding modules 
---------------------------------------------

<img width="1061" alt="Screenshot 2023-03-06 at 5 51 50 AM" src="https://user-images.githubusercontent.com/105562242/222994699-90c00899-d84b-4672-8de3-853767ec56f6.png">

<img width="1088" alt="Screenshot 2023-03-06 at 5 52 51 AM" src="https://user-images.githubusercontent.com/105562242/222994755-88466785-ddc5-40d9-8fc3-7a484aea9abd.png">

<img width="1243" alt="Screenshot 2023-03-06 at 5 54 58 AM" src="https://user-images.githubusercontent.com/105562242/222994880-ea43ea1c-5bba-4786-97a1-c91a96143688.png">

<img width="1179" alt="Screenshot 2023-03-06 at 5 55 10 AM" src="https://user-images.githubusercontent.com/105562242/222994897-99a1aa37-b16b-4f24-8ee8-0f2d2c853504.png">

<img width="1249" alt="Screenshot 2023-03-06 at 6 00 31 AM" src="https://user-images.githubusercontent.com/105562242/222995235-32c438a9-9acc-4db4-97ff-8f628a669d0d.png">

<img width="1082" alt="Screenshot 2023-03-06 at 6 01 50 AM" src="https://user-images.githubusercontent.com/105562242/222995308-eb632944-b369-42ea-b40f-8371bae09b0f.png">

```tar -zxvf nginx-1.23.3.tar.gz ``` 

##### We have to run ./configure script to check all necessay tools are installed.To install compiler and all necessary tools : ```tar -zxvf nginx-1.23.3.tar.gz``` need to run. Post we will run ./configure script again to check all are good. 



<img width="740" alt="Screenshot 2023-03-06 at 6 07 54 AM" src="https://user-images.githubusercontent.com/105562242/222995659-44bf75b6-4de3-4fd3-a1fc-e31bf7fc8485.png">

<img width="1087" alt="Screenshot 2023-03-06 at 6 06 19 AM" src="https://user-images.githubusercontent.com/105562242/222995559-56dc6017-4d12-419d-8dce-4f745722fe1a.png">



##### Below libary is missing. we will install it. ```apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev``` We will run again ./configure script and we will see all good.



<img width="860" alt="Screenshot 2023-03-06 at 6 10 48 AM" src="https://user-images.githubusercontent.com/105562242/222995862-60342105-9f63-49a7-94d3-654100184e0a.png">

<img width="1089" alt="Screenshot 2023-03-06 at 6 14 12 AM" src="https://user-images.githubusercontent.com/105562242/222996123-5ac223aa-4f67-492a-9f7f-e2762cbd163f.png">



##### Please refer :- https://nginx.org/en/docs/configure.html. we will configure paths. 



``` ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module```

<img width="1091" alt="Screenshot 2023-03-06 at 6 23 39 AM" src="https://user-images.githubusercontent.com/105562242/222996775-783ec8d4-1c0d-40b3-aba9-ce6d1c0602fc.png">

<img width="1018" alt="Screenshot 2023-03-06 at 6 23 59 AM" src="https://user-images.githubusercontent.com/105562242/222996800-e71e1d15-a829-4e41-bf92-6fffa3470ed9.png">


##### Now we will ```make``` and also run ```make install```



<img width="1083" alt="Screenshot 2023-03-06 at 6 27 04 AM" src="https://user-images.githubusercontent.com/105562242/222997007-5465958e-a38d-4f71-8052-dff1a7ac28e3.png">

<img width="1088" alt="Screenshot 2023-03-06 at 6 28 20 AM" src="https://user-images.githubusercontent.com/105562242/222997111-6ad30e38-59ac-44c6-a7cf-3842ff751cc6.png">



##### To verify we will check version and configuratin files under /etc/nginx. We will check nginx running or not post that.



<img width="942" alt="Screenshot 2023-03-06 at 6 30 12 AM" src="https://user-images.githubusercontent.com/105562242/222997271-93096442-8c14-42e2-a6d7-5439b28261a2.png">

<img width="1087" alt="Screenshot 2023-03-06 at 6 31 24 AM" src="https://user-images.githubusercontent.com/105562242/222997342-4cc9959e-b931-48d2-912c-4fdf5b30d504.png">



### Adding nginx service :

##### To stop nginx service :

<img width="1087" alt="Screenshot 2023-03-06 at 6 39 23 AM" src="https://user-images.githubusercontent.com/105562242/222997859-836271a7-a18c-4c8e-a996-d96f557e716a.png">

<img width="1086" alt="Screenshot 2023-03-06 at 6 39 40 AM" src="https://user-images.githubusercontent.com/105562242/222997893-a9e80846-2d59-4daa-aee1-b441484387a4.png">



### We will add systemd service ( First we need to check if we are able to control nginx service with systemctl command. if we are able to control systemctl command then we can skip this part. we have to excute below steps in-case we are not able to control nginx service with systemctl command). To enable nginx service during boot. We have to enable with systemctl command ```systemctl enable nginx```
--------------------------------------------------------------------


https://www.freedesktop.org/wiki/Software/systemd/

https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/

https://www.nginx.com/resources/wiki/start/topics/examples/systemd/

##### We will create and go to path /lib/systemd/system/nginx.service and do changes services paths as below

<img width="846" alt="Screenshot 2023-03-06 at 6 48 20 AM" src="https://user-images.githubusercontent.com/105562242/222998671-672ebe79-f018-435a-b4b4-ccf249fdcfcf.png">


<img width="1087" alt="Screenshot 2023-03-06 at 6 46 29 AM" src="https://user-images.githubusercontent.com/105562242/222998495-fe0de508-4d14-4d83-b730-6c7c84de1e8c.png">

<img width="1072" alt="Screenshot 2023-03-06 at 7 02 30 AM" src="https://user-images.githubusercontent.com/105562242/222999978-6d054169-9c81-47cc-9b2a-93ee56273758.png">

