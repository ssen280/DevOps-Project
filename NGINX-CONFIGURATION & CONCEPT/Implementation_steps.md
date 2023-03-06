


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
