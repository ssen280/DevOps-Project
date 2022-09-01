
#### MIGRATION TO THE CLOUD WITH CONTAINERIZATION - DOCKER
---------------------------------------------------------------------
#### INTRODUCTION
In this project, the frontend and the backend(MySQL) of tooling application is built and containerized using DOCKER of which its image is pushed to Docker registry. And further in the project, the php-todo application is also built into a container and pushed into the AWS Elastic Container Registry using a CI/CD tool known as Jenkins and Docker Compose is also implemented.

The following outlines the steps:


#### STEP 1: Creating MySQL Container For Tooling App Backend :
---------------------------------------------------------------------------
1. We will install Docker on our Ubuntu system. Please follow below link 
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
2. We will custom network with a subnet dedicated for both MySQL and the Tooling application so that they connect
``` $ docker network create --subnet=172.18.0.0/24 tooling_app_network ```
3. We will create environment variable to store sql root password and database schema to commect tooling application. we will add to /etc/environment file to keep it parmanently.

<img width="994" alt="Screenshot 2022-09-01 at 2 34 08 PM" src="https://user-images.githubusercontent.com/105562242/187876179-eb829963-c611-4e3a-b5aa-cb04a067c420.png">

4. We will Pull the MySQL image and running the container with our custom created docker network.
``` docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW -d mysql/mysql-server:latest ```

5. Because it's not a good practice to connect to MySQL server remotely using the root user. Creating a file create_user.sql and adding the following code in order to create a user:
``` CREATE USER 'saikat'@'%' IDENTIFIED BY 'password123'; GRANT ALL PRIVILEGES ON * . * TO 'saikat'@'%';```

6. Running the script to create the new user 
``` $ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql```

<img width="1452" alt="Screenshot 2022-08-31 at 1 27 51 PM" src="https://user-images.githubusercontent.com/105562242/187877900-15c9426c-b3f2-4954-8cc9-1d9d184a314e.png">

7. Connecting to the MySQL server from a second container running the MySQL client utility
``` $ docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u -p ```

 <img width="1352" alt="Screenshot 2022-08-31 at 1 09 09 PM" src="https://user-images.githubusercontent.com/105562242/187878499-a3cc2c80-5d16-4dca-8544-ca4a0eaa4fd4.png">
8. Cloning the Tooling-app repository: 
```$ git clone https://github.com/darey-devops/tooling.git```

9. Exporting the location of the SQL file that contains data for setting up the MySQL database. In step 3 we have already added environment variables.

10. Using the SQL script to create the database and prepare the schema
``` $ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema ```

11. From the tooling app directory where the Dockerfile is, we will edit it and add database details.

12. Dockerfile details :

<img width="1045" alt="Screenshot 2022-09-01 at 2 57 45 PM" src="https://user-images.githubusercontent.com/105562242/187881059-ac355d67-3201-4a44-81c4-d42d02e7f718.png">

13. We will build docker image. we have run docker buid command where Docerfile is available. In are case its in tooling app directory.
<img width="1014" alt="Screenshot 2022-08-31 at 1 11 44 PM" src="https://user-images.githubusercontent.com/105562242/187881514-7ac3fb4f-c7c7-4fa6-b231-3c40f5c6132d.png">

14. We will run docker image. 
<img width="1451" alt="Screenshot 2022-08-31 at 1 12 24 PM" src="https://user-images.githubusercontent.com/105562242/187881793-58696743-302b-4848-ba46-03253fda61cf.png">

15. We will access tooling web page

16. Here we can HTTP requiest to get the web page

<img width="1451" alt="Screenshot 2022-08-31 at 1 12 24 PM" src="https://user-images.githubusercontent.com/105562242/187882087-bc54a1f4-7a9f-4b66-80a1-7f8cf5231100.png">
<img width="1518" alt="Screenshot 2022-08-31 at 1 12 13 PM" src="https://user-images.githubusercontent.com/105562242/187882117-00dbabcb-615f-48be-b901-2ca6f86c31ca.png">

