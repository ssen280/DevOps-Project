
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

### Migrating PHP-Todo App Into A Containerized Application
------------------------------------------------------------------
1. Clone repo from https://github.com/darey-devops/php-todo.git
2. We will write Dockerfile in todo app folder
<img width="1006" alt="Screenshot 2022-09-01 at 3 19 27 PM" src="https://user-images.githubusercontent.com/105562242/187885613-ce8c4a5c-134c-452a-bee9-3210af51782e.png">
3. We will create MySQL container for the php-todo frontend
<img width="1452" alt="Screenshot 2022-08-31 at 1 50 02 PM" src="https://user-images.githubusercontent.com/105562242/187886020-acd6990c-b61d-4f7e-bb28-77fe9bfa9279.png">
4. We will update .env file in todo app folder before running docker build command to build docker image

```
APP_ENV=local
APP_DEBUG=true
APP_KEY=SomeRandomString
APP_URL=http://localhost

DB_HOST=mysqlserverhost
DB_DATABASE=homestead
DB_USERNAME=saikat
DB_PASSWORD=password123

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

```

5. We will run build command to create the Docker image of the app
<img width="969" alt="Screenshot 2022-08-31 at 1 30 02 PM" src="https://user-images.githubusercontent.com/105562242/187887343-8a27b4f3-252d-465c-bd8d-acd48066ef3b.png">

6. We will run the container 
``` Docker run —network tooling_app_network -p 8090:5001 -it php-todo:0.0.3 ```

7. Here we can see we are getting error from todo webpage
<img width="1312" alt="Screenshot 2022-08-31 at 1 52 41 PM" src="https://user-images.githubusercontent.com/105562242/187889578-57523577-16e4-463d-a1a1-0c34c4991827.png">

8. We will create homestead database in sql which is associated to todo app
9. Then we will run Php artisan migrate command in todo app container

<img width="892" alt="Screenshot 2022-08-31 at 3 44 58 PM" src="https://user-images.githubusercontent.com/105562242/187899255-b9148069-abb8-40a3-8091-b6f2745ddc32.png">

10. we are able to access todo app now
<img width="1428" alt="Screenshot 2022-08-31 at 4 07 24 PM" src="https://user-images.githubusercontent.com/105562242/187899364-32c5389b-f11c-4ef5-8224-b66a751e4485.png">

11. We will push todo docker image to docker repo. First we will create accunt on docker site. and creat a new repo and give it a name. then we will login to docker on system from were we want to push image. 

<img width="1428" alt="Screenshot 2022-08-31 at 4 07 24 PM" src="https://user-images.githubusercontent.com/105562242/187899722-f7d04f56-3bbf-4072-98d1-86141ce5cfc9.png">

12. We will use docker tag and docker push command to upload image to docker repo

<img width="1428" alt="Screenshot 2022-08-31 at 4 07 24 PM" src="https://user-images.githubusercontent.com/105562242/187900781-2ef1f98c-c605-4e00-96ad-e63450c46ea2.png">
<img width="1404" alt="Screenshot 2022-09-01 at 4 43 32 PM" src="https://user-images.githubusercontent.com/105562242/187900866-75b2fa2f-26d9-40c8-811b-d02759f04866.png">

#### Running Docker Build And Docker Push on Jenkins
--------------------------------------------------------------
1. We will install Jenkin on our ubuntu docker server. We will follow below link to install jenkins

https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-20-04

2. We will setup Nexus repo server on ubuntu as seperate server. We will follow below link to istall Nexus. Latest download link

https://download.sonatype.com/nexus/3/nexus-3.41.1-01-unix.tar.gz
https://www.fosstechnix.com/how-to-install-nexus-repository-on-ubuntu/

3. We will open 8080, 8081, 8083 ports to security group which both jenkin/docker server and nexus server both are using. 
4. We will install docker and nexus plugins to Jenkin. 

<img width="1409" alt="Screenshot 2022-09-07 at 12 17 53 AM" src="https://user-images.githubusercontent.com/105562242/188714876-bd45de00-ca1e-4376-94ef-fe1b68f9b811.png">

<img width="1378" alt="Screenshot 2022-09-07 at 12 18 17 AM" src="https://user-images.githubusercontent.com/105562242/188714958-7dfbfc99-f189-4a70-b7c3-ae59aaa9a401.png">

5. We will configure nexus in jenkins and test the connection.

<img width="1241" alt="Screenshot 2022-09-07 at 12 23 32 AM" src="https://user-images.githubusercontent.com/105562242/188715902-883f87d1-8913-453e-914b-572c99219f60.png">

6. We will create docker repo to Nexus and use port 8083 for this particular repo

<img width="1329" alt="Screenshot 2022-09-07 at 12 29 52 AM" src="https://user-images.githubusercontent.com/105562242/188717050-83d793a8-bc06-4010-97a2-bf715ba4f924.png">

<img width="1724" alt="Screenshot 2022-09-07 at 12 30 19 AM" src="https://user-images.githubusercontent.com/105562242/188717131-9262225a-6782-4e4d-ac0e-d70f5adf6b07.png">

7. We will create two branches in my php-todo github repo: develop and feature
8. We will create Jenkinsfile for the two branches which will run docker build and push the image to my nexus docker repo.

```
pipeline {
    agent any

    environment {
        imageName = "php-todo"
        registryCredentials = "nexus-new"
        registry = "54.208.101.39:8083"
        dockerImage = ''
    }

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/ssen280/php-todo.git'
      }
    }
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build imageName
        }
      }
    }
    // Uploading Docker images into Nexus Registry
    stage('Uploading to Nexus') {
     steps{
         script {
             docker.withRegistry( 'http://'+registry, registryCredentials ) {
             dockerImage.push('feature-latest')
          }
        }
      }
    }
  }
}

```

9. For feature branch we will use dockerImage.push('feature-latest') and for developer branch it would dockerImage.push('develop-latest')
10. Nexus run on http and docker dont accept http request, docker only accpet https request.
11. We will create daemon.json file in /etc/docker on docker server. we will configure below lines to daemon.json file

<img width="649" alt="Screenshot 2022-09-07 at 12 39 06 AM" src="https://user-images.githubusercontent.com/105562242/188719021-64edad36-e80e-40a3-98a3-4aa6215e2230.png">

12. We will create a multibranch pipeline job and linking it to the php-todo repository

<img width="917" alt="Screenshot 2022-09-06 at 2 50 16 PM" src="https://user-images.githubusercontent.com/105562242/188719469-ac03f7da-24fc-4ece-bde8-df72fd356322.png">

13. We will run pipeline job

<img width="1352" alt="Screenshot 2022-09-07 at 12 43 08 AM" src="https://user-images.githubusercontent.com/105562242/188719615-a4843321-0239-4ad0-91ea-b69c9cf3cba7.png">

 <img width="1681" alt="Screenshot 2022-09-07 at 12 44 03 AM" src="https://user-images.githubusercontent.com/105562242/188719780-b3f2f3b7-d61e-42c1-8a37-081e8936ba60.png">
 
<img width="1692" alt="Screenshot 2022-09-07 at 12 44 34 AM" src="https://user-images.githubusercontent.com/105562242/188719857-f188c466-d58b-476e-9937-3dcb400e6386.png">

<img width="1399" alt="Screenshot 2022-09-07 at 12 45 15 AM" src="https://user-images.githubusercontent.com/105562242/188719961-6f0e9d35-37f5-48b9-a80e-607b3a07cfaa.png">

<img width="1431" alt="Screenshot 2022-09-07 at 12 45 38 AM" src="https://user-images.githubusercontent.com/105562242/188720017-f593ac9f-fa32-4703-ab2c-314aeeb9a896.png">

14. Here we can docker image uploaded to nexus repo

<img width="1725" alt="Screenshot 2022-09-07 at 12 48 05 AM" src="https://user-images.githubusercontent.com/105562242/188720432-b8951778-e8ac-4eab-ad97-b7187794546c.png">

<img width="1727" alt="Screenshot 2022-09-07 at 12 48 33 AM" src="https://user-images.githubusercontent.com/105562242/188720509-61424843-7b59-4dce-8f73-c291e5ce5bf7.png">

#### Running Docker Compose
---------------------------------------

Docker Compose takes care of the hard work involved in running Docker commands on the terminal to create an image, launch an application inside it and get the applications up and running by writing a declarative code in YAML which gets all the applications and dependencies up and running with minimal effort by launching a single command.

Refactoring the Tooling app POC in order to leverage the power of Docker Compose.
Creating a file called tooling.yaml
Writing the Docker Compose in yaml syntax, defining services, networks, and volumes:

```
version: "3.3"
services:
  tooling_frontend:
    build: .
    depends_on:
      - db
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: toolingdb
      MYSQL_USER: saikat
      MYSQL_PASSWORD: password123
      MYSQL_ROOT_PASSWORD: password123  
    volumes:
      - db:/var/lib/mysql
      - .datadump.sql/:/docker-entrypoint-initdb.d/datadump.sql

networks:
  default:
    external:
      name: tooling_app_network
  
volumes:
  tooling_frontend:
  db:
```

<img width="656" alt="Screenshot 2022-09-07 at 1 30 59 AM" src="https://user-images.githubusercontent.com/105562242/188728349-1defbd36-96bf-4485-a571-016ba0af16a9.png">
