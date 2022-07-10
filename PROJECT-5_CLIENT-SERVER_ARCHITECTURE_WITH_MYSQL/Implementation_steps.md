
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
##### For fresh installations of MySQL, you’ll want to run the DBMS’s included security script. This script changes some of the less secure default options for things like remote root logins and sample users. We will run below command 
```
sudo mysql_secure_installation
```

##### This will take us through a series of prompts where we can make some changes to MySQL installation’s security options. The first prompt will ask whether we’d like to set up the Validate Password Plugin, which can be used to test the password strength of new MySQL users before deeming them valid.

##### If we elect to set up the Validate Password Plugin, any MySQL user we create that authenticates with a password will be required to have a password that satisfies the policy we select. The strongest policy level — which you can select by entering 2 — will require passwords to be at least eight characters long and include a mix of uppercase, lowercase, numeric, and special characters:

<img width="1194" alt="Screenshot 2022-07-10 at 1 25 11 PM" src="https://user-images.githubusercontent.com/105562242/178136278-52d10f16-6b31-4203-9d63-3567f2aa15ca.png">

##### After setting root user passoword we will access mysql with root user using below commnad 
` mysql -u root -p `

##### We will create new user so that uer can remotely access mysql from client 

` CREATE USER 'saikat'@'172.31.30.217' IDENTIFIED BY 'password';`

<img width="946" alt="Screenshot 2022-07-10 at 1 37 42 PM" src="https://user-images.githubusercontent.com/105562242/178136650-22d35104-3ef0-4f14-a746-f2c0ae290dd0.png">

##### We will give appropriate access to user. We will run below commands
```
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'saikat'@'172.31.30.217' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
##### Now we will install sql client on client server

```
sudo apt install mysql-client
```
##### We have to enable port 3306 on databaser server with client server ip address so that client server access mysql remotely.

<img width="928" alt="Screenshot 2022-07-10 at 1 45 22 PM" src="https://user-images.githubusercontent.com/105562242/178136914-52cf17c3-2c4d-4cd1-8c69-648d566f6b1a.png">

##### We have to replace ‘127.0.0.1’ to ‘0.0.0.0’ in /etc/mysql/mysql.conf.d/mysqld.cnf path on database server 


<img width="1136" alt="Screenshot 2022-07-10 at 1 47 06 PM" src="https://user-images.githubusercontent.com/105562242/178136975-03072d1e-4255-4ae4-84b5-565a9efd9f4c.png">

##### Now we will access mysql with user which we have created for remote access. 

```
mysql -u USERNAME -p PASSWORD -h HOST-OR-SERVER-IP
```

<img width="1093" alt="Screenshot 2022-07-10 at 1 53 34 PM" src="https://user-images.githubusercontent.com/105562242/178137169-2fe041dd-c3ce-4d7a-81d8-45cffa699fa4.png">



