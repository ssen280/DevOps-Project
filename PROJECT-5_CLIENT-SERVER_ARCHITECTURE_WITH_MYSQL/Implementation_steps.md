
##### CLIENT-SERVER ARCHITECTURE WITH MYSQL :

###### Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.
###### In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server".
###### A simple diagram of Web Client-Server architecture is presented below:

<img width="1020" alt="Screenshot 2022-07-09 at 10 04 47 PM" src="https://user-images.githubusercontent.com/105562242/178114754-3f650e39-17f9-401f-83b0-91319b5f59da.png">

<img width="1114" alt="Screenshot 2022-07-09 at 10 06 19 PM" src="https://user-images.githubusercontent.com/105562242/178114808-6f7acbb2-2468-4165-89c5-b73e18c8e980.png">

##### First we will create two linux virtual servers. One is for database server and another one is for client. 

##### We will install mysql server on database server using below commands
```
sudo apt update
sudo apt install mysql-server
sudo systemctl start mysql.service
sudo systemctl start mysql.service
```
