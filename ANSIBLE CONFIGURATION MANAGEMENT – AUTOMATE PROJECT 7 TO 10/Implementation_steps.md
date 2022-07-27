## PROJECT 11
In Projects 7 to 10 we had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy  web application.

In this project we will automate manual jobs by using Ansible 

#### Ansible Client as a Jump Server (Bastion Host)
##### A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If we think about the current architecture we are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

##### On the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

<img width="895" alt="Screenshot 2022-07-26 at 8 27 49 PM" src="https://user-images.githubusercontent.com/105562242/181040109-38c8aa60-9f5f-4b7b-aa36-f830a78b3640.png">

##### When we reach Project 15, we will see a Bastion host in proper action. But for now, we will develop Ansible scripts to simulate the use of a Jump box/Bastion host to access our Web Servers.

#### Task:
##### Install and configure Ansible client to act as a Jump Server/Bastion Host
##### Create a simple Ansible playbook to automate servers configuration

#### INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE :

##### We will use our jenkin server from project-9 as Ansible management server and rename it as Jenkins-Ansible.
##### We will install Ansible 
```
sudo apt update

sudo apt install ansible
```
##### We will create github repo for ansible and name it ansible-configuration-management and we will configure webhuk to it to automate junkin build job
<img width="821" alt="Screenshot 2022-07-26 at 8 36 01 PM" src="https://user-images.githubusercontent.com/105562242/181041910-6dddddb3-0d08-4fe5-9297-211613d56062.png">

##### We will configure Jenkins build job to save our repository content every time we change it. We will follow below steps to do this :

 ##### 1.Create a new Freestyle project ansible in Jenkins and point it to ‘ansible-config-mgt’ repository.
 ##### 2.Configure Webhook in GitHub and set webhook to trigger ansible build.
 ##### 3.Configure a Post-build job to save all (**) files.
 ##### 4. We will add elastic IP to jenkin-ansible server so that pubpic IP does not change after shutdown/power up
 
 <img width="1601" alt="Screenshot 2022-07-26 at 9 24 22 PM" src="https://user-images.githubusercontent.com/105562242/181052808-0acd094c-b569-4173-a2f6-e5b1be3a1ad6.png">

<img width="752" alt="Screenshot 2022-07-26 at 9 24 42 PM" src="https://user-images.githubusercontent.com/105562242/181052883-4263b21c-efbf-4a19-beba-ff10574cf54a.png">


<img width="1125" alt="Screenshot 2022-07-26 at 9 25 14 PM" src="https://user-images.githubusercontent.com/105562242/181053009-dc0b617a-f497-431a-a56a-9cac12b91960.png">

<img width="1402" alt="Screenshot 2022-07-26 at 9 28 02 PM" src="https://user-images.githubusercontent.com/105562242/181053667-91afd76a-003e-4eb8-916f-462b87996b63.png">

##### Now our setup will look as below:

<img width="1081" alt="Screenshot 2022-07-26 at 9 33 07 PM" src="https://user-images.githubusercontent.com/105562242/181054786-f65a0093-8d92-495f-9ded-ffa6a64d0ca3.png">

#### Step 2 – Prepare your development environment using Visual Studio Code.

##### We will access our servers from VS code so that we can manage github repo and update it, edit it, create new files/folder, push it to github 

<img width="1022" alt="Screenshot 2022-07-25 at 4 00 59 PM" src="https://user-images.githubusercontent.com/105562242/181056503-d9151fb3-39a1-433c-aece-285ef66f8445.png">

##### We will create new branch under ansible-configuration-management and give it a meaningfull name so that we can understand it what we are actually going to do

##### 1.We will checkout the newly created feature branch to local machine and start building our code and directory structure
##### 2.We will create a directory and name it playbooks – it will be used to store all your playbook files.
##### 3.We will create a directory and name it inventory – it will be used to keep your hosts organised.
##### 4.Within the playbooks folder, we will create first playbook, and name it common.yml
##### 5.Within the inventory folder, we will create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

<img width="1080" alt="Screenshot 2022-07-25 at 4 23 10 PM" src="https://user-images.githubusercontent.com/105562242/181058537-8eb8fac3-3f5f-4b44-a3db-171e80850353.png">


##### Step 4 – Set up an Ansible Inventory:
##### An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

##### We will configure our invertory file which is inventory/dev.yml with our nfs, database, loadbalancer and web servers 

<img width="824" alt="Screenshot 2022-07-25 at 4 28 21 PM" src="https://user-images.githubusercontent.com/105562242/181059999-04ad8f57-6ea8-4cd5-9f41-e4aea40b1854.png">

##### Ansible connect hosts with ssh protocol. hence we will configure below so that we able to push Ansible configuration from ansible management server with is jenkin-ansible sever to rest of servers ( database, nfs, loadbalancer, web servers)

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>

# Then we will check if key has been added with the command below

ssh-add -l

# Now we will ssh to our jenkin-ansible server.

ssh -A ubuntu@public-ip-of jenkin-ansibel server

# Now from Jenkin-ansible server we will do password less ssh to rest of servers 

```

##### We will push all files folders from local repo to remote repo and it will trigger jenkin build automatically

<img width="1610" alt="Screenshot 2022-07-25 at 4 51 37 PM" src="https://user-images.githubusercontent.com/105562242/181061274-309cc84d-065e-4660-8367-11a73c54bcbd.png">

<img width="1111" alt="Screenshot 2022-07-25 at 4 56 38 PM" src="https://user-images.githubusercontent.com/105562242/181061410-4f64174b-ddeb-4abf-88d8-268c8a9f3599.png">

<img width="1160" alt="Screenshot 2022-07-25 at 4 56 54 PM" src="https://user-images.githubusercontent.com/105562242/181061500-d3b2f598-3f65-4251-bfd9-943d36faf67c.png">

<img width="1727" alt="Screenshot 2022-07-25 at 4 57 31 PM" src="https://user-images.githubusercontent.com/105562242/181061600-9f757f04-56e4-4f02-b812-91eb17318054.png">

#### CREATE PLAYBOOK :

#### First we will configure connection-check playbook so that we can check the rechability from ansible management server to rest of servers. As we know Ansible by-defualt get server details from host file which is located to /etc/ansible path. So here we just populated host file and run this playbook.

<img width="799" alt="Screenshot 2022-07-26 at 10 07 29 PM" src="https://user-images.githubusercontent.com/105562242/181062048-60e2b463-72b0-4a42-b2d9-35fd77dc7557.png">
<img width="976" alt="Screenshot 2022-07-25 at 6 17 15 PM" src="https://user-images.githubusercontent.com/105562242/181062116-18c877da-e959-49a8-bc6e-b55ab09f6212.png">

#### Now we will do ping by giving absolute path of dev file to check ping response and also run connection check playback with dev inventrory file. 

<img width="1160" alt="Screenshot 2022-07-25 at 8 33 27 PM" src="https://user-images.githubusercontent.com/105562242/181063157-3659d413-06cf-4323-b198-56525b3cff3a.png">

<img width="1233" alt="Screenshot 2022-07-25 at 8 34 48 PM" src="https://user-images.githubusercontent.com/105562242/181063313-f7cfe84c-069d-4b72-afa9-1a50cae19393.png">

##### Now we will create two playbook files. one is common.yaml and another one is nginx-common.yaml
 ##### 1. We will configure common.yaml to install wireshark on all server and for this playbook, ansible get host details from default host file
 ##### 2. We will create common-nginx.yaml to install apache and httpd on all servers and for this playbook, ansibel get host details from inventory/dev.yaml
 ##### 3. We will check if all servers are updated
 
 <img width="982" alt="Screenshot 2022-07-26 at 10 22 38 PM" src="https://user-images.githubusercontent.com/105562242/181064993-04deb458-2cc6-4e11-b8bd-305dc5935aaa.png">

 <img width="1437" alt="Screenshot 2022-07-25 at 6 31 02 PM" src="https://user-images.githubusercontent.com/105562242/181065080-8bc5e2b8-393f-4361-8de5-55780e61eab3.png">

 <img width="1006" alt="Screenshot 2022-07-26 at 10 24 16 PM" src="https://user-images.githubusercontent.com/105562242/181065296-c46ceea0-8b8a-45d5-9d94-314ecdb39b62.png">


 <img width="1678" alt="Screenshot 2022-07-25 at 8 45 34 PM" src="https://user-images.githubusercontent.com/105562242/181066549-c1bcf97e-08a9-4f98-a79d-a8adb974f666.png">


<img width="1680" alt="Screenshot 2022-07-25 at 8 48 23 PM" src="https://user-images.githubusercontent.com/105562242/181066657-d2d461f5-c66b-412b-a541-9f3592c599dc.png">

<img width="1614" alt="Screenshot 2022-07-25 at 10 54 01 PM" src="https://user-images.githubusercontent.com/105562242/181066688-1bc2775a-c132-45ea-8992-4c74b16e9796.png">


<img width="1724" alt="Screenshot 2022-07-26 at 4 47 04 AM" src="https://user-images.githubusercontent.com/105562242/181066952-3b0ee4d4-f826-4907-9697-c8b3f5e13064.png">


<img width="1200" alt="Screenshot 2022-07-26 at 4 48 30 AM" src="https://user-images.githubusercontent.com/105562242/181067413-6232d392-af8b-42d9-ba73-0aa7ef81df93.png">

<img width="1027" alt="Screenshot 2022-07-27 at 7 33 34 AM" src="https://user-images.githubusercontent.com/105562242/181144511-be14340c-4ced-4220-afe0-7c302fab087a.png">



