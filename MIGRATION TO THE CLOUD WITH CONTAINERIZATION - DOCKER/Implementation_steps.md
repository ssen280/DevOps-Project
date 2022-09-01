
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
