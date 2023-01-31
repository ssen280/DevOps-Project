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

<img width="939" alt="Screenshot 2023-01-31 at 10 03 49 AM" src="https://user-images.githubusercontent.com/105562242/215725057-828d3d7d-b446-4346-86b1-33d0901d933c.png">

Now we will update RDS endpoints to tooling and wordpress ansible db configuration code file. user and password we have to use from terraform.auto.tfvars.

<img width="642" alt="Screenshot 2023-01-31 at 3 20 16 PM" src="https://user-images.githubusercontent.com/105562242/215726407-df98399b-8ac5-45a4-9250-e124bb50bfc0.png">


<img width="1398" alt="Screenshot 2023-01-31 at 10 18 18 AM" src="https://user-images.githubusercontent.com/105562242/215725306-a90d7a80-b4e7-4374-9fff-9325108949bd.png">

<img width="1144" alt="Screenshot 2023-01-31 at 3 16 46 PM" src="https://user-images.githubusercontent.com/105562242/215725537-1c28a3da-7a56-4d39-b1c7-ecba15927cc3.png">

<img width="1179" alt="Screenshot 2023-01-31 at 3 17 26 PM" src="https://user-images.githubusercontent.com/105562242/215725697-e4fdc284-25a8-4090-8e23-72a6a1ffc8f7.png">

Now we have to update EFS endpoints to respective wordpress and tooling asible code file 

<img width="1725" alt="Screenshot 2023-01-31 at 10 19 55 AM" src="https://user-images.githubusercontent.com/105562242/215726752-c8458144-1681-4dab-bbaa-d4be72545515.png">

<img width="1657" alt="Screenshot 2023-01-31 at 10 24 31 AM" src="https://user-images.githubusercontent.com/105562242/215726827-eaefc9e6-710a-4b44-b2a0-faacf0b8eb55.png">

<img width="1647" alt="Screenshot 2023-01-31 at 10 26 29 AM" src="https://user-images.githubusercontent.com/105562242/215726862-3a44e841-52eb-4f21-8c9f-e6c832f09fec.png">

Now we will update internal elb to nginx aisible code file 

<img width="1444" alt="Screenshot 2023-01-31 at 10 19 16 AM" src="https://user-images.githubusercontent.com/105562242/215727939-3b191591-cdf5-49c9-9755-67b9c4032ef5.png">

<img width="1163" alt="Screenshot 2023-01-31 at 3 26 59 PM" src="https://user-images.githubusercontent.com/105562242/215728025-a2a4f322-0924-4080-b874-05e742df9628.png">


Now we will run ```Ansiblel$ ansible-playbook -i inventory/aws_ec2.yml playbooks/site. yml``` to do installation

<img width="1110" alt="Screenshot 2023-01-31 at 10 44 24 AM" src="https://user-images.githubusercontent.com/105562242/215727365-cc5748a8-f98d-4860-a2c1-71497802ddc8.png">

Here we can see we are getting error while installing Install PyMySQL for wordpress and tooling.
```
[ec2-user@ip-10-0-4-40 Ansible]$ ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml 
[DEPRECATION WARNING]: [defaults]callback_whitelist option, normalizing names to new standard, use callbacks_enabled 
instead. This feature will be removed from ansible-core in version 2.15. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [tag_Name_ACS_nginx] ********************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
Tuesday 31 January 2023  05:13:54 +0000 (0:00:00.024)       0:00:00.024 ******* 
ok: [10.0.2.232]

PLAY [tag_Name_ACS_nginx] ********************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
Tuesday 31 January 2023  05:13:57 +0000 (0:00:03.683)       0:00:03.707 ******* 
ok: [10.0.2.232]

TASK [nginx : install nginx on the webserver] ************************************************************************
Tuesday 31 January 2023  05:13:58 +0000 (0:00:01.074)       0:00:04.781 ******* 
changed: [10.0.2.232]

TASK [nginx : ensure nginx is started and enabled] *******************************************************************
Tuesday 31 January 2023  05:14:03 +0000 (0:00:04.740)       0:00:09.522 ******* 
changed: [10.0.2.232]

TASK [nginx : create html directory] *********************************************************************************
Tuesday 31 January 2023  05:14:04 +0000 (0:00:01.425)       0:00:10.947 ******* 
changed: [10.0.2.232]

TASK [nginx : rename the defualt configurarion file] *****************************************************************
Tuesday 31 January 2023  05:14:05 +0000 (0:00:00.683)       0:00:11.631 ******* 
changed: [10.0.2.232]

TASK [nginx : create new nginx file] *********************************************************************************
Tuesday 31 January 2023  05:14:06 +0000 (0:00:00.689)       0:00:12.320 ******* 
changed: [10.0.2.232]

TASK [nginx : configure nginx config file] ***************************************************************************
Tuesday 31 January 2023  05:14:06 +0000 (0:00:00.500)       0:00:12.820 ******* 
changed: [10.0.2.232]

PLAY [tag_Name_ACS_tooling] ******************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
Tuesday 31 January 2023  05:14:08 +0000 (0:00:01.280)       0:00:14.101 ******* 
ok: [10.0.1.94]

PLAY [tag_Name_ACS_tooling] ******************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
Tuesday 31 January 2023  05:14:11 +0000 (0:00:02.899)       0:00:17.001 ******* 
ok: [10.0.1.94]

TASK [tooling : mounting share(s)] ***********************************************************************************
Tuesday 31 January 2023  05:14:12 +0000 (0:00:01.095)       0:00:18.096 ******* 
changed: [10.0.1.94]

TASK [tooling : install httpd on the webserver] **********************************************************************
Tuesday 31 January 2023  05:14:17 +0000 (0:00:04.999)       0:00:23.096 ******* 
changed: [10.0.1.94]

TASK [tooling : install mod_ssl on the webserver] ********************************************************************
Tuesday 31 January 2023  05:14:21 +0000 (0:00:03.943)       0:00:27.039 ******* 
ok: [10.0.1.94]

TASK [tooling : install PHP] *****************************************************************************************
Tuesday 31 January 2023  05:14:22 +0000 (0:00:01.544)       0:00:28.583 ******* 
changed: [10.0.1.94]

TASK [tooling : ensure php-fpm is started and enabled] ***************************************************************
Tuesday 31 January 2023  05:14:27 +0000 (0:00:04.710)       0:00:33.294 ******* 
changed: [10.0.1.94]

TASK [tooling : Clone the repository] ********************************************************************************
Tuesday 31 January 2023  05:14:28 +0000 (0:00:01.318)       0:00:34.612 ******* 
changed: [10.0.1.94]

TASK [tooling : copy the html from tooling to /var/www/html] *********************************************************
Tuesday 31 January 2023  05:14:30 +0000 (0:00:01.467)       0:00:36.080 ******* 
changed: [10.0.1.94]

TASK [tooling : create healthstatus file] ****************************************************************************
Tuesday 31 January 2023  05:14:31 +0000 (0:00:00.994)       0:00:37.075 ******* 
changed: [10.0.1.94]

TASK [tooling : Allow apache to modify /var/www/html] ****************************************************************
Tuesday 31 January 2023  05:14:31 +0000 (0:00:00.634)       0:00:37.709 ******* 
changed: [10.0.1.94]

TASK [tooling : Install PyMySQL] *************************************************************************************
Tuesday 31 January 2023  05:14:35 +0000 (0:00:03.463)       0:00:41.173 ******* 
fatal: [10.0.1.94]: FAILED! => {"changed": false, "msg": "Unable to find any of pip3 to use.  pip needs to be installed."}

RUNNING HANDLER [tooling : Restart httpd] ****************************************************************************
Tuesday 31 January 2023  05:14:36 +0000 (0:00:00.867)       0:00:42.040 ******* 

RUNNING HANDLER [tooling : Restart php-fpm] **************************************************************************
Tuesday 31 January 2023  05:14:36 +0000 (0:00:00.000)       0:00:42.041 ******* 

PLAY RECAP ***********************************************************************************************************
10.0.1.94                  : ok=11   changed=8    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
10.0.2.232                 : ok=8    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Tuesday 31 January 2023  05:14:36 +0000 (0:00:00.004)       0:00:42.045 ******* 
=============================================================================== 
tooling : mounting share(s) ----------------------------------------------------------------------------------- 5.00s
nginx : install nginx on the webserver ------------------------------------------------------------------------ 4.74s
tooling : install PHP ----------------------------------------------------------------------------------------- 4.71s
tooling : install httpd on the webserver ---------------------------------------------------------------------- 3.94s
Gathering Facts ----------------------------------------------------------------------------------------------- 3.68s
tooling : Allow apache to modify /var/www/html ---------------------------------------------------------------- 3.46s
Gathering Facts ----------------------------------------------------------------------------------------------- 2.90s
tooling : install mod_ssl on the webserver -------------------------------------------------------------------- 1.54s
tooling : Clone the repository -------------------------------------------------------------------------------- 1.47s
nginx : ensure nginx is started and enabled ------------------------------------------------------------------- 1.43s
tooling : ensure php-fpm is started and enabled --------------------------------------------------------------- 1.32s
nginx : configure nginx config file --------------------------------------------------------------------------- 1.28s
Gathering Facts ----------------------------------------------------------------------------------------------- 1.10s
Gathering Facts ----------------------------------------------------------------------------------------------- 1.07s
tooling : copy the html from tooling to /var/www/html --------------------------------------------------------- 0.99s
tooling : Install PyMySQL ------------------------------------------------------------------------------------- 0.87s
nginx : rename the defualt configurarion file ----------------------------------------------------------------- 0.69s
nginx : create html directory --------------------------------------------------------------------------------- 0.68s
tooling : create healthstatus file ---------------------------------------------------------------------------- 0.63s
nginx : create new nginx file --------------------------------------------------------------------------------- 0.50s
[ec2-user@ip-10-0-4-40 Ansible]$ 
[ec2-user@ip-10-0-4-40 Ansible]$ 
[ec2-user@ip-10-0-4-40 Ansible]$ 
```
To fix this error we will add below two lines of code to asible code 

```
- name: Install pip 
  yum:
   name: python3-pip 
   state: present

```

<img width="582" alt="Screenshot 2023-01-31 at 12 23 02 PM" src="https://user-images.githubusercontent.com/105562242/215728994-8994f5eb-a613-40aa-b54d-ae94ce794b88.png">

Now we will run Ansible again and we will see there is no error 

```
[ec2-user@ip-10-0-4-40 Ansible]$ ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml
[DEPRECATION WARNING]: [defaults]callback_whitelist option, normalizing names to new standard, use 
callbacks_enabled instead. This feature will be removed from ansible-core in version 2.15. Deprecation warnings
 can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [tag_Name_ACS_wordpress] **********************************************************************************

TASK [Gathering Facts] *****************************************************************************************
Tuesday 31 January 2023  06:28:33 +0000 (0:00:00.017)       0:00:00.017 ******* 
ok: [10.0.3.159]

PLAY [tag_Name_ACS_wordpress] **********************************************************************************

TASK [Gathering Facts] *****************************************************************************************
Tuesday 31 January 2023  06:28:35 +0000 (0:00:01.723)       0:00:01.740 ******* 
ok: [10.0.3.159]

TASK [wordpress : mounting share(s)] ***************************************************************************
Tuesday 31 January 2023  06:28:36 +0000 (0:00:01.087)       0:00:02.828 ******* 
ok: [10.0.3.159]

TASK [wordpress : install httpd on the webserver] **************************************************************
Tuesday 31 January 2023  06:28:36 +0000 (0:00:00.788)       0:00:03.617 ******* 
ok: [10.0.3.159]

TASK [wordpress : install PHP] *********************************************************************************
Tuesday 31 January 2023  06:28:38 +0000 (0:00:01.755)       0:00:05.373 ******* 
ok: [10.0.3.159]

TASK [wordpress : ensure php-fpm is started and enabled] *******************************************************
Tuesday 31 January 2023  06:28:40 +0000 (0:00:01.580)       0:00:06.953 ******* 
changed: [10.0.3.159]

TASK [wordpress : Download wordpress compressed file] **********************************************************
Tuesday 31 January 2023  06:28:41 +0000 (0:00:01.251)       0:00:08.204 ******* 
ok: [10.0.3.159]

TASK [wordpress : unzip the compressed file] *******************************************************************
Tuesday 31 January 2023  06:28:43 +0000 (0:00:02.038)       0:00:10.243 ******* 
ok: [10.0.3.159]

TASK [wordpress : Copy config.php] *****************************************************************************
Tuesday 31 January 2023  06:28:49 +0000 (0:00:06.282)       0:00:16.526 ******* 
ok: [10.0.3.159]

TASK [wordpress : deploy the code] *****************************************************************************
Tuesday 31 January 2023  06:28:50 +0000 (0:00:00.793)       0:00:17.319 ******* 
ok: [10.0.3.159]

TASK [wordpress : create healthstatus file] ********************************************************************
Tuesday 31 January 2023  06:28:55 +0000 (0:00:05.183)       0:00:22.503 ******* 
changed: [10.0.3.159]

TASK [wordpress : Allow apache to modify /var/www/html] ********************************************************
Tuesday 31 January 2023  06:28:56 +0000 (0:00:00.803)       0:00:23.307 ******* 
ok: [10.0.3.159]

TASK [wordpress : install httpd on the webserver] **************************************************************
Tuesday 31 January 2023  06:28:57 +0000 (0:00:01.078)       0:00:24.385 ******* 
ok: [10.0.3.159]

TASK [wordpress : install mod_ssl on the webserver] ************************************************************
Tuesday 31 January 2023  06:28:59 +0000 (0:00:01.572)       0:00:25.958 ******* 
ok: [10.0.3.159]

TASK [wordpress : install PHP] *********************************************************************************
Tuesday 31 January 2023  06:29:00 +0000 (0:00:01.624)       0:00:27.582 ******* 
ok: [10.0.3.159]

TASK [wordpress : ensure php-fpm is started and enabled] *******************************************************
Tuesday 31 January 2023  06:29:02 +0000 (0:00:01.578)       0:00:29.161 ******* 
changed: [10.0.3.159]

TASK [wordpress : Install pip] *********************************************************************************
Tuesday 31 January 2023  06:29:03 +0000 (0:00:00.917)       0:00:30.078 ******* 
changed: [10.0.3.159]

TASK [wordpress : Install PyMySQL] *****************************************************************************
Tuesday 31 January 2023  06:29:07 +0000 (0:00:03.988)       0:00:34.067 ******* 
changed: [10.0.3.159]

TASK [wordpress : create database] *****************************************************************************
Tuesday 31 January 2023  06:29:09 +0000 (0:00:01.931)       0:00:35.998 ******* 
changed: [10.0.3.159]

TASK [wordpress : Input wordpress credentials] *****************************************************************
Tuesday 31 January 2023  06:29:10 +0000 (0:00:00.836)       0:00:36.835 ******* 
ok: [10.0.3.159] => (item={'regexp': '^localhost', 'line': 'terraform-20230131035310046700000010.cnsdq4a7pewm.us-east-1.rds.amazonaws.com'})
ok: [10.0.3.159] => (item={'regexp': '^username_here', 'line': 'saikat'})
ok: [10.0.3.159] => (item={'regexp': '^database_name_here', 'line': 'wordpressdb'})
ok: [10.0.3.159] => (item={'regexp': '^password_here', 'line': 'devopspblproject'})

RUNNING HANDLER [wordpress : Restart httpd] ********************************************************************
Tuesday 31 January 2023  06:29:12 +0000 (0:00:02.608)       0:00:39.443 ******* 
changed: [10.0.3.159]

PLAY RECAP *****************************************************************************************************
10.0.3.159                 : ok=21   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Tuesday 31 January 2023  06:29:14 +0000 (0:00:02.009)       0:00:41.453 ******* 
=============================================================================== 
wordpress : unzip the compressed file ------------------------------------------------------------------- 6.28s
wordpress : deploy the code ----------------------------------------------------------------------------- 5.18s
wordpress : Install pip --------------------------------------------------------------------------------- 3.99s
wordpress : Input wordpress credentials ----------------------------------------------------------------- 2.61s
wordpress : Download wordpress compressed file ---------------------------------------------------------- 2.04s
wordpress : Restart httpd ------------------------------------------------------------------------------- 2.01s
wordpress : Install PyMySQL ----------------------------------------------------------------------------- 1.93s
wordpress : install httpd on the webserver -------------------------------------------------------------- 1.76s
Gathering Facts ----------------------------------------------------------------------------------------- 1.72s
wordpress : install mod_ssl on the webserver ------------------------------------------------------------ 1.62s
wordpress : install PHP --------------------------------------------------------------------------------- 1.58s
wordpress : install PHP --------------------------------------------------------------------------------- 1.58s
wordpress : install httpd on the webserver -------------------------------------------------------------- 1.57s
wordpress : ensure php-fpm is started and enabled ------------------------------------------------------- 1.25s
Gathering Facts ----------------------------------------------------------------------------------------- 1.09s
wordpress : Allow apache to modify /var/www/html -------------------------------------------------------- 1.08s
wordpress : ensure php-fpm is started and enabled ------------------------------------------------------- 0.92s
wordpress : create database ----------------------------------------------------------------------------- 0.84s
wordpress : create healthstatus file -------------------------------------------------------------------- 0.80s
wordpress : Copy config.php ----------------------------------------------------------------------------- 0.79s
[ec2-user@ip-10-0-4-40 Ansible]$ 
```
```
[ec2-user@ip-10-0-4-40 Ansible]$ ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml
[DEPRECATION WARNING]: [defaults]callback_whitelist option, normalizing names to new standard, use 
callbacks_enabled instead. This feature will be removed from ansible-core in version 2.15. Deprecation warnings
 can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [tag_Name_ACS_wordpress] **********************************************************************************

TASK [Gathering Facts] *****************************************************************************************
Tuesday 31 January 2023  06:35:08 +0000 (0:00:00.016)       0:00:00.016 ******* 
ok: [10.0.3.159]

PLAY [tag_Name_ACS_tooling] ************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
Tuesday 31 January 2023  06:35:09 +0000 (0:00:01.712)       0:00:01.729 ******* 
ok: [10.0.1.94]

TASK [tooling : mounting share(s)] *****************************************************************************
Tuesday 31 January 2023  06:35:11 +0000 (0:00:01.249)       0:00:02.979 ******* 
ok: [10.0.1.94]

TASK [tooling : install httpd on the webserver] ****************************************************************
Tuesday 31 January 2023  06:35:11 +0000 (0:00:00.795)       0:00:03.774 ******* 
ok: [10.0.1.94]

TASK [tooling : install mod_ssl on the webserver] **************************************************************
Tuesday 31 January 2023  06:35:13 +0000 (0:00:01.748)       0:00:05.522 ******* 
ok: [10.0.1.94]

TASK [tooling : install PHP] ***********************************************************************************
Tuesday 31 January 2023  06:35:15 +0000 (0:00:01.533)       0:00:07.056 ******* 
ok: [10.0.1.94]

TASK [tooling : ensure php-fpm is started and enabled] *********************************************************
Tuesday 31 January 2023  06:35:16 +0000 (0:00:01.537)       0:00:08.594 ******* 
changed: [10.0.1.94]

TASK [tooling : Clone the repository] **************************************************************************
Tuesday 31 January 2023  06:35:17 +0000 (0:00:01.247)       0:00:09.841 ******* 
ok: [10.0.1.94]

TASK [tooling : copy the html from tooling to /var/www/html] ***************************************************
Tuesday 31 January 2023  06:35:18 +0000 (0:00:01.048)       0:00:10.889 ******* 
ok: [10.0.1.94]

TASK [tooling : create healthstatus file] **********************************************************************
Tuesday 31 January 2023  06:35:19 +0000 (0:00:00.826)       0:00:11.715 ******* 
changed: [10.0.1.94]

TASK [tooling : Allow apache to modify /var/www/html] **********************************************************
Tuesday 31 January 2023  06:35:20 +0000 (0:00:00.796)       0:00:12.512 ******* 
ok: [10.0.1.94]

TASK [tooling : Install pip] ***********************************************************************************
Tuesday 31 January 2023  06:35:21 +0000 (0:00:01.074)       0:00:13.586 ******* 
changed: [10.0.1.94]

TASK [tooling : Install PyMySQL] *******************************************************************************
Tuesday 31 January 2023  06:35:25 +0000 (0:00:03.560)       0:00:17.146 ******* 
changed: [10.0.1.94]

TASK [tooling : create database] *******************************************************************************
Tuesday 31 January 2023  06:35:27 +0000 (0:00:01.914)       0:00:19.061 ******* 
changed: [10.0.1.94]

TASK [tooling : Input tooling credentials] *********************************************************************
Tuesday 31 January 2023  06:35:27 +0000 (0:00:00.828)       0:00:19.889 ******* 
ok: [10.0.1.94] => (item={'regexp': '^mysql.tooling.svc.cluster.local', 'line': 'terraform-20230131035310046700000010.cnsdq4a7pewm.us-east-1.rds.amazonaws.com'})
ok: [10.0.1.94] => (item={'regexp': '^admin', 'line': 'saikat'})
ok: [10.0.1.94] => (item={'regexp': '^tooling', 'line': 'toolingdb'})
ok: [10.0.1.94] => (item={'regexp': '^admin', 'line': 'devopspblproject'})

TASK [tooling : ensure httpd is started and enabled] ***********************************************************
Tuesday 31 January 2023  06:35:30 +0000 (0:00:02.589)       0:00:22.479 ******* 
ok: [10.0.1.94]

RUNNING HANDLER [tooling : Restart php-fpm] ********************************************************************
Tuesday 31 January 2023  06:35:31 +0000 (0:00:00.817)       0:00:23.297 ******* 
changed: [10.0.1.94]

PLAY RECAP *****************************************************************************************************
10.0.1.94                  : ok=16   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.3.159                 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Tuesday 31 January 2023  06:35:32 +0000 (0:00:00.919)       0:00:24.216 ******* 
=============================================================================== 
tooling : Install pip ----------------------------------------------------------------------------------- 3.56s
tooling : Input tooling credentials --------------------------------------------------------------------- 2.59s
tooling : Install PyMySQL ------------------------------------------------------------------------------- 1.91s
tooling : install httpd on the webserver ---------------------------------------------------------------- 1.75s
Gathering Facts ----------------------------------------------------------------------------------------- 1.71s
tooling : install PHP ----------------------------------------------------------------------------------- 1.54s
tooling : install mod_ssl on the webserver -------------------------------------------------------------- 1.53s
Gathering Facts ----------------------------------------------------------------------------------------- 1.25s
tooling : ensure php-fpm is started and enabled --------------------------------------------------------- 1.25s
tooling : Allow apache to modify /var/www/html ---------------------------------------------------------- 1.07s
tooling : Clone the repository -------------------------------------------------------------------------- 1.05s
tooling : Restart php-fpm ------------------------------------------------------------------------------- 0.92s
tooling : create database ------------------------------------------------------------------------------- 0.83s
tooling : copy the html from tooling to /var/www/html --------------------------------------------------- 0.83s
tooling : ensure httpd is started and enabled ----------------------------------------------------------- 0.82s
tooling : create healthstatus file ---------------------------------------------------------------------- 0.80s
tooling : mounting share(s) ----------------------------------------------------------------------------- 0.80s
[ec2-user@ip-10-0-4-40 Ansible]$ 
```
Please login to tooling,wordpress & nginx servers and make sure httpds and nginx serveces are running fine. 

Here we are able to access tooling site. but wordpress is giving error

<img width="1206" alt="Screenshot 2023-01-31 at 11 31 03 AM" src="https://user-images.githubusercontent.com/105562242/215729691-82a7aa48-17e6-47ad-880f-d4e2743de8ef.png">

<img width="1177" alt="Screenshot 2023-01-31 at 12 13 40 PM" src="https://user-images.githubusercontent.com/105562242/215729749-4d38656d-7cdc-42ca-93f7-e8a0e880b304.png">

We will check wordpress configuration and fix this db error issue. To fix this we have to update usename/password/hostame/db name to wordpress configuration as below 

<img width="962" alt="Screenshot 2023-01-31 at 12 14 56 PM" src="https://user-images.githubusercontent.com/105562242/215730006-7da0be01-4161-4a78-8617-0e8a90983ab0.png">

<img width="1024" alt="Screenshot 2023-01-31 at 12 17 52 PM" src="https://user-images.githubusercontent.com/105562242/215730071-4daf6a36-4bbc-4735-82ed-4cf871a20e81.png">

Now we are able to access wordpress without any error

<img width="1258" alt="Screenshot 2023-01-31 at 12 20 42 PM" src="https://user-images.githubusercontent.com/105562242/215730204-175fb498-2ed3-487d-9987-14acc33ba2a2.png">
