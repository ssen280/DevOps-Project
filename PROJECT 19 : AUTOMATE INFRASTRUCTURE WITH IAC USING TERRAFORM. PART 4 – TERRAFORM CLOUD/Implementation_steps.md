#### Automate Infrastructure With IaC using Terraform. Part 4 - Terraform Cloud.
------------------------------------------------------------------------------------

In this project we will be using Packer to build our images, and Ansible to configure the infrastructure, so for that we are going to make few changes to our our existing repository from Project 18.

The files that would be addedd are:

* AMI: for building packer images
* Ansible: for Ansible scripts to configure the infrastructure

##### Action Plan for project 19

* Build images using packer
* confirm the AMIs in the console
* update terrafrom script with new ami IDs generated from packer build
* create terraform cloud account and backend
* run terraform script
* update ansible script with values from teraform output
  * RDS endpoints for wordpress and tooling
  * Database name, password and username for wordpress and tooling
  * Access point ID for wordpress and tooling
  * Internal load balancee DNS for nginx reverse proxy
* run ansible script
* check the website

We will install packer on our system and create AMIs. We have have to make sure we are using correct owners who owns relevent images. 
###### Please refer below. Here we can see owner 309956199498 refers to Redhat and thy own Redhat images.We will also make sure we are using correct image name which we can get from Amazon market place.

##### Packer code : 

<img width="879" alt="Screenshot 2023-01-28 at 10 43 59 AM" src="https://user-images.githubusercontent.com/105562242/215243450-8808e956-179a-49d3-b0cf-9dba3c73e685.png">

<img width="742" alt="Screenshot 2023-01-28 at 10 45 01 AM" src="https://user-images.githubusercontent.com/105562242/215243500-41e9413c-4012-473e-aa64-bb3e7cc9b648.png">

<img width="1012" alt="Screenshot 2023-01-28 at 10 46 14 AM" src="https://user-images.githubusercontent.com/105562242/215243543-e6c79d98-77d1-4036-800e-a6df504fb2fe.png">



