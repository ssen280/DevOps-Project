#### In this project we will continue working with ansible-configuration-management repository and make some improvements of our code. Now we need to refactor our Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows us to organize your tasks and reuse them when needed.

##### Code Refactoring:
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

 In our case, we will move things around a little bit in the code, but the overal state of the infrastructure remains the same.
 Let see how We can improve our Ansible code!

#### Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.

##### 1. We will create a folder name ansible-config-artifact on our jenkin-ansible server
##### 2. We will change permission of the folder/direcory so that ansible can access it
##### 3. on jenkin web console we will install copy artifact plugin from available plugin
##### 4. We will create new free style project and name is save_artifacts
##### 5. This project will be triggered by completion of your existing ansible project.

###### Note: We can configure number of builds to keep in order to save space on the server, for example, we might want to keep only last 2 or 5 build results. we can also make this change to your ansible job.

##### The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

```
sudo mkdir /home/ubuntu/ansible-config-artifact
chmod -R 0777 /home/ubuntu/ansible-config-artifact

```


<img width="873" alt="Screenshot 2022-07-27 at 9 19 02 AM" src="https://user-images.githubusercontent.com/105562242/181156662-c58ecc97-387b-43e3-980f-fae2bfb97e89.png">

<img width="1587" alt="Screenshot 2022-07-27 at 9 15 52 AM" src="https://user-images.githubusercontent.com/105562242/181156342-bb6ecd99-88a1-479b-ae1c-7f4392e892e0.png">
<img width="1148" alt="Screenshot 2022-07-27 at 9 16 23 AM" src="https://user-images.githubusercontent.com/105562242/181156400-7ddc3805-8756-4508-9075-bc9ad80a33a1.png">
<img width="1134" alt="Screenshot 2022-07-27 at 9 16 52 AM" src="https://user-images.githubusercontent.com/105562242/181156456-3c041c9e-23f1-4cfe-a78f-53d3647c249f.png">

<img width="1726" alt="Screenshot 2022-07-26 at 3 53 45 AM" src="https://user-images.githubusercontent.com/105562242/181158547-4a835375-7228-4a49-aab2-1b2363ec4071.png">

<img width="1723" alt="Screenshot 2022-07-26 at 4 11 28 AM" src="https://user-images.githubusercontent.com/105562242/181158591-b9ae1bfc-ba10-4c28-ba8b-a1b7f99f85e8.png">

<img width="970" alt="Screenshot 2022-07-26 at 4 11 45 AM" src="https://user-images.githubusercontent.com/105562242/181158646-ec38e0bf-b407-499f-a473-c7cf300980a2.png">

<img width="1723" alt="Screenshot 2022-07-26 at 5 02 38 AM" src="https://user-images.githubusercontent.com/105562242/181158832-7c9b30d9-2cfd-4ae7-99a1-dfc06fb267c8.png">

<img width="1724" alt="Screenshot 2022-07-26 at 5 02 54 AM" src="https://user-images.githubusercontent.com/105562242/181159107-aebf5515-f6bd-46ac-aa42-a67451754ee5.png">

#### Let see code re-use in action by importing other playbooks.

##### 1.Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that we created previously. 

##### 2.Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of our work. It is not an Ansible specific concept, therefore we can choose how we want to organize our work. we will see why the folder name has a prefix of static very soon. For now, just follow along.

##### 3.Move common.yml file into the newly created static-assignments folder.

##### 4.Inside site.yml file, import common.yml playbook.


```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

##### 5.The code above uses built in import_playbook Ansible module. our folder structure should look like this

```
├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
```
##### 6. We will create one playbook file common-del.yml to uinstall wireshark from severs and we will configure site.yaml file with common-del.yml to unstall wireshark and we will check connectiity first as well using check-connection.yaml

<img width="869" alt="Screenshot 2022-07-27 at 9 31 27 AM" src="https://user-images.githubusercontent.com/105562242/181158109-bc2b0b8b-f6a3-4d3f-abca-603ba263c04a.png">

<img width="689" alt="Screenshot 2022-07-27 at 9 31 59 AM" src="https://user-images.githubusercontent.com/105562242/181158168-2473b817-e950-4f81-a810-6e02ecf0950b.png">

<img width="1365" alt="Screenshot 2022-07-26 at 5 17 38 AM" src="https://user-images.githubusercontent.com/105562242/181159283-147dfa7c-8f1a-4d90-8188-25fb984d3bb1.png">

<img width="1365" alt="Screenshot 2022-07-26 at 5 21 13 AM" src="https://user-images.githubusercontent.com/105562242/181159341-351fca3b-d33b-42a2-9a61-e3ef92bc1aed.png">

<img width="965" alt="Screenshot 2022-07-26 at 5 23 36 AM" src="https://user-images.githubusercontent.com/105562242/181159379-2cc7c129-f478-4de4-abd5-18687e6cceae.png">

<img width="867" alt="Screenshot 2022-07-26 at 5 24 19 AM" src="https://user-images.githubusercontent.com/105562242/181159553-00a88e4d-9f12-4490-892b-b672c4942753.png">

#### Configure UAT Webservers with a role ‘Webserver’
We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.
Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

##### To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory. We will use below steps
```
# Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (we need to create roles directory upfront)

mkdir roles
cd roles
ansible-galaxy init webserver

# The entire folder structure should look like below

└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
        
# We will remove unnecessary files and folders for it. finally folder structure would be as below under roles

└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates


```
##### We will update inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of 2 UAT Web servers
<img width="957" alt="Screenshot 2022-07-26 at 6 33 27 AM" src="https://user-images.githubusercontent.com/105562242/181161112-9dac2608-94c2-49dc-b3bb-96e1e60c56f0.png">

<img width="493" alt="Screenshot 2022-07-27 at 9 53 43 AM" src="https://user-images.githubusercontent.com/105562242/181161158-d9d2c78b-6194-444b-ac43-836099b5419b.png">

<img width="1365" alt="Screenshot 2022-07-26 at 6 39 39 AM" src="https://user-images.githubusercontent.com/105562242/181161199-11646645-4afb-48fd-ba06-387619de441a.png">

##### In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to our roles directory roles_path so Ansible could know where to find configured roles.

<img width="714" alt="Screenshot 2022-07-26 at 6 47 16 AM" src="https://user-images.githubusercontent.com/105562242/181161573-9f410a41-e89d-45d1-a828-8f2512a5db59.png">

##### It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

##### Install and configure Apache (httpd service)
##### Clone Tooling website from GitHub repo.
##### Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
##### Make sure httpd service is started
##### main.yml may consist of following tasks:
  
<img width="932" alt="Screenshot 2022-07-27 at 10 01 07 AM" src="https://user-images.githubusercontent.com/105562242/181161963-e70711ef-5704-44ae-9732-45785af34dde.png">

##### We will ceate new assignment for uat-webservers as uat-webservers.yml under static assignet. 
<img width="434" alt="Screenshot 2022-07-27 at 10 07 50 AM" src="https://user-images.githubusercontent.com/105562242/181162590-b1f156f5-6563-430e-8c53-3c421f535a21.png">

##### Remember that the entry point to our ansible configuration is the site.yml file. Therefore, we need to refer our uat-webservers.yml role inside site.yml.

<img width="624" alt="Screenshot 2022-07-27 at 10 09 46 AM" src="https://user-images.githubusercontent.com/105562242/181162788-c5e438c8-f5e3-4c9d-8215-af6420d8dcf0.png">


##### We will run playbook now

<img width="1678" alt="Screenshot 2022-07-26 at 7 13 08 AM" src="https://user-images.githubusercontent.com/105562242/181162832-d1bdafc9-6233-49f4-a28c-c52db2d705ef.png">
<img width="960" alt="Screenshot 2022-07-26 at 7 15 40 AM" src="https://user-images.githubusercontent.com/105562242/181162861-75b45b2f-63cd-49d9-9233-b473fb03a8a7.png">

<img width="1054" alt="Screenshot 2022-07-26 at 7 16 57 AM" src="https://user-images.githubusercontent.com/105562242/181166825-ac622b62-c405-467e-8520-2c361ad7a85e.png">


<img width="1349" alt="Screenshot 2022-07-26 at 7 21 04 AM" src="https://user-images.githubusercontent.com/105562242/181162949-49118599-e1dd-4e62-8b7a-de239c71c6cb.png">
<img width="1233" alt="Screenshot 2022-07-26 at 7 21 15 AM" src="https://user-images.githubusercontent.com/105562242/181162967-c056d161-1489-4483-a012-8424767d64ba.png">

<img width="1233" alt="Screenshot 2022-07-26 at 7 21 15 AM" src="https://user-images.githubusercontent.com/105562242/181166968-209b0f87-509e-4020-9e4f-299f95173eaf.png">

<img width="847" alt="Screenshot 2022-07-27 at 10 46 35 AM" src="https://user-images.githubusercontent.com/105562242/181167093-3878fd7b-a3b2-497f-995c-490e434cfcc0.png">
<img width="1715" alt="Screenshot 2022-07-27 at 12 02 57 PM" src="https://user-images.githubusercontent.com/105562242/181177651-48bd0490-2859-4693-82c0-c37199f848b5.png">

