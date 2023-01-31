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
##### Please refer below. Here we can see owner 309956199498 refers to Redhat and thy own Redhat images.We will also make sure we are using correct image name which we can get from Amazon market place.

##### Packer code : 

<img width="879" alt="Screenshot 2023-01-28 at 10 43 59 AM" src="https://user-images.githubusercontent.com/105562242/215243450-8808e956-179a-49d3-b0cf-9dba3c73e685.png">

<img width="742" alt="Screenshot 2023-01-28 at 10 45 01 AM" src="https://user-images.githubusercontent.com/105562242/215243500-41e9413c-4012-473e-aa64-bb3e7cc9b648.png">

<img width="1012" alt="Screenshot 2023-01-28 at 10 46 14 AM" src="https://user-images.githubusercontent.com/105562242/215243543-e6c79d98-77d1-4036-800e-a6df504fb2fe.png">

We will follow same and launch few more AMIs

<img width="1016" alt="Screenshot 2023-01-28 at 10 48 32 AM" src="https://user-images.githubusercontent.com/105562242/215243915-98157b59-566e-4ed4-9e11-355ef0eaff67.png">


<img width="1016" alt="Screenshot 2023-01-28 at 10 54 46 AM" src="https://user-images.githubusercontent.com/105562242/215246270-4cf50ee3-00ec-4ebf-b999-d99e687ffa8a.png">


<img width="1017" alt="Screenshot 2023-01-28 at 10 56 21 AM" src="https://user-images.githubusercontent.com/105562242/215246981-2e0f1629-3f7e-4da0-8dde-eea4b49b093b.png">

<img width="1186" alt="Screenshot 2023-01-28 at 11 02 37 AM" src="https://user-images.githubusercontent.com/105562242/215247973-7569c91c-b86c-4997-9f7e-7548c4eb4d50.png">

<img width="1018" alt="Screenshot 2023-01-28 at 11 05 35 AM" src="https://user-images.githubusercontent.com/105562242/215248073-806e2451-858a-4d64-9d16-7148cdea095c.png">

<img width="982" alt="Screenshot 2023-01-28 at 11 12 52 AM" src="https://user-images.githubusercontent.com/105562242/215248419-98699114-233c-44a0-a053-fd0b4bc2c633.png">


Update the new AMI's ID from the packer build in the terraform script

<img width="1488" alt="Screenshot 2023-01-28 at 11 17 32 AM" src="https://user-images.githubusercontent.com/105562242/215248564-031fc833-0d49-4b64-87f2-b8cb8be7ab36.png">


<img width="862" alt="Screenshot 2023-01-30 at 7 01 49 PM" src="https://user-images.githubusercontent.com/105562242/215491227-2983b9b5-6dd9-4f0e-b984-f49aff119704.png">


Now we will create terraform cloud account and backend. We will connect our github repo (containing the terraform script) with the cloud account and this will create a workspace on the account, which is where all terraform plan, apply, destroy and other commands will be executed. The workspace ensures that any changes made in the repo will be noticed by terraform cloud and it will run the code which is more like our running terraform plan and if we want to apply it, we will run it on the terraform cloud UI.

##### GitHub Repo for terraform code : https://github.com/ssen280/TERRAFORM-PBL19

<img width="1425" alt="Screenshot 2023-01-30 at 7 05 09 PM" src="https://user-images.githubusercontent.com/105562242/215491914-e24a2323-411b-47a1-a863-733861579cdf.png">

<img width="1724" alt="Screenshot 2023-01-30 at 6 59 22 PM" src="https://user-images.githubusercontent.com/105562242/215492008-fdafae26-9569-42b2-a82f-eed84e44b53e.png">

<img width="1444" alt="Screenshot 2023-01-30 at 6 59 39 PM" src="https://user-images.githubusercontent.com/105562242/215492036-45e283ba-29f6-48a9-bcfb-ba9ddaa0dc8c.png">

<img width="1444" alt="Screenshot 2023-01-30 at 6 59 54 PM" src="https://user-images.githubusercontent.com/105562242/215492092-8ed8899b-902d-4571-8bfd-5f8018887ff6.png">

<img width="1420" alt="Screenshot 2023-01-30 at 7 00 09 PM" src="https://user-images.githubusercontent.com/105562242/215492110-682d3ab9-ebd2-4408-a139-c08311df01e7.png">

<img width="1401" alt="Screenshot 2023-01-30 at 7 00 24 PM" src="https://user-images.githubusercontent.com/105562242/215492142-104e67d2-4ae8-47fd-b7c6-6b37ea39b4b2.png">

<img width="1420" alt="Screenshot 2023-01-30 at 7 00 40 PM" src="https://user-images.githubusercontent.com/105562242/215492163-96bfe111-f2ec-436d-abbb-362fdc452a44.png">

<img width="1486" alt="Screenshot 2023-01-30 at 7 10 43 PM" src="https://user-images.githubusercontent.com/105562242/215493448-766851da-cdea-4856-a91a-37dc93537b33.png">

<img width="1411" alt="Screenshot 2023-01-30 at 7 11 15 PM" src="https://user-images.githubusercontent.com/105562242/215493511-28a655d7-bcf9-4986-9d4c-0c9235b68516.png">

<img width="1455" alt="Screenshot 2023-01-30 at 7 11 28 PM" src="https://user-images.githubusercontent.com/105562242/215493539-3407cde6-bb6b-4517-ae3c-d132469079af.png">

<img width="1288" alt="Screenshot 2023-01-30 at 7 13 15 PM" src="https://user-images.githubusercontent.com/105562242/215493742-0c3498c0-f31c-45b0-b1a5-99dbff3bfd03.png">


On our workspace, the states files created when an apply is made on the terraform script is kept on the account compared to having it locally and the backend.

Run apply on the terraform script via the account UI.

After the apply is run, ensure that all resources are created as expected.

SSH into the Bastion instance and clone https://github.com/Taiwolawal/ansible-deploy-pbl-19.git which contains Ansible scripts which will be used to configure the infrastructure as required.

To ensure the Ansible file can get all the required information from our AWS account such as our instance IP addresses, tags for each instances, we need to run aws configure and enter all required credentials.

Ensure that we have ssh-agent enabled on our bastion instance, so that we can easily SSH into Nginx and Webservers.

Update the ansible script with values such as:

##### GitHub link : https://github.com/ssen280/PBL-project-19/tree/main/Ansible

* RDS endpoints for wordpress and tooling
* Database name, password and username for wordpress and tooling
* Access point ID for wordpress and tooling
* Internal load balancee DNS for nginx reverse proxy

We will check if we are able to access rest of servers from jump server using ssh 

<img width="1020" alt="Screenshot 2023-01-31 at 9 32 40 AM" src="https://user-images.githubusercontent.com/105562242/215722600-b0c9a9a7-a2c6-4d89-982d-fc9bd86ee7b1.png">

<img width="885" alt="Screenshot 2023-01-31 at 9 32 56 AM" src="https://user-images.githubusercontent.com/105562242/215722657-25dad568-19df-440f-a43d-2fdea703afa8.png">

We will do AWS configuration on jump server with our access details :

<img width="1018" alt="Screenshot 2023-01-31 at 9 38 59 AM" src="https://user-images.githubusercontent.com/105562242/215723809-209f4792-3f93-4ba1-969e-c7073eab9717.png">

Now We will check if Ansible able to fetch all ec2 hosts 

`ansible-inventory -i inventory/aws_ec2.ym --graph`

Here we can see we are getting error. we will fix it by installing boto3


<img width="1098" alt="Screenshot 2023-01-31 at 10 02 08 AM" src="https://user-images.githubusercontent.com/105562242/215724550-689aad29-ce55-4bed-b7a9-8637606ed352.png">

<img width="1098" alt="Screenshot 2023-01-31 at 10 02 30 AM" src="https://user-images.githubusercontent.com/105562242/215724707-58287931-0eca-4d81-9de1-6b1986bb29b0.png">


<img width="1098" alt="Screenshot 2023-01-31 at 10 02 49 AM" src="https://user-images.githubusercontent.com/105562242/215724723-9f9771bb-059e-458f-ad5d-50920be0b09d.png">

<img width="1098" alt="Screenshot 2023-01-31 at 10 03 10 AM" src="https://user-images.githubusercontent.com/105562242/215724780-22bb3411-b8ee-4779-be16-3e518f8b540a.png">

<img width="965" alt="Screenshot 2023-01-31 at 10 03 21 AM" src="https://user-images.githubusercontent.com/105562242/215724829-64bdb21c-16f2-48c4-9766-a654cb54964f.png">

<img width="1308" alt="Screenshot 2023-01-31 at 10 03 42 AM" src="https://user-images.githubusercontent.com/105562242/215724871-f259b2f4-d27e-4d77-8c6d-b78b424efaa7.png">

