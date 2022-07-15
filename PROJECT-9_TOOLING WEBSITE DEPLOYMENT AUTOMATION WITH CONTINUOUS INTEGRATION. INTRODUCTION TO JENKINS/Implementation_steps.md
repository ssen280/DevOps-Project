
In previous Project 8 we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and you have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers – it is not a big deal to configure them manually. Imagine that you would need to repeat the same task over and over again adding dozens or even hundreds of servers.

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server – Jenkins. It is one of the mostl popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<yourname>/tooling will be automatically be updated to the Tooling Website.
  
Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how our updated architecture will look like upon competion of this project:
  
<img width="1116" alt="Screenshot 2022-07-15 at 3 18 31 PM" src="https://user-images.githubusercontent.com/105562242/179199214-96f74a8e-f1cc-4d49-8091-dcb7ff1f22e5.png">

Step 1 – Install Jenkins server
Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

Install JDK (since Jenkins is a Java-based application)
  
```
sudo apt update
sudo apt install default-jdk-headless
Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
 ```
Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```
By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
  
<img width="1257" alt="Screenshot 2022-07-15 at 4 42 10 PM" src="https://user-images.githubusercontent.com/105562242/179212412-e684afd3-7cae-468e-a070-25628ca47f82.png">

Perform initial Jenkins setup.
From browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

We will be prompted to provide a default admin password

Retrieve it from  server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Then we will be asked which plugings to install – choose suggested plugins.

Once plugins installation is done – create an admin user and you will get your Jenkins server address.
  
  <img width="1155" alt="Screenshot 2022-07-15 at 4 45 13 PM" src="https://user-images.githubusercontent.com/105562242/179212837-3948c92b-96a1-4f88-821b-b0fd451532fa.png">

 Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
 
This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

Enable webhooks in your GitHub repository settings
  
Go to Jenkins web console, click "New Item" and create a "Freestyle project"

To connect GitHub repository, we will need to provide its URL, we can copy from the repository itself
  
![image](https://user-images.githubusercontent.com/105562242/179213256-ed9be603-f1e3-4f91-8e02-7d855234e90b.png)
![image](https://user-images.githubusercontent.com/105562242/179213306-b9316921-b0d0-4c74-b7ef-8f27bee20ca0.png)
![image](https://user-images.githubusercontent.com/105562242/179213337-fb98e888-e463-4a8e-aa43-520e481b781b.png)
  
In configuration of our Jenkins freestyle project choose Git repository, provide there the link to our Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![image](https://user-images.githubusercontent.com/105562242/179213460-a55a044c-1862-4139-a623-e2e7ae9b048a.png)

Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and we will see it under #1
  
![image](https://user-images.githubusercontent.com/105562242/179213562-c934978e-6623-422a-9cf3-1169694b408b.png)
We can open the build and check in "Console Output" if it has run successfully.

If so – congratulations! We have just made our very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

Click "Configure" job/project and add these two configurations
Configure triggering the job from GitHub webhook:

 ![image](https://user-images.githubusercontent.com/105562242/179213761-40be4877-9c0e-4919-9e3e-794d7ec2c194.png)

Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".
  
![image](https://user-images.githubusercontent.com/105562242/179213826-c2baf0ea-1424-42be-bf04-06267407c2a2.png)

![image](https://user-images.githubusercontent.com/105562242/179213866-f74b77c9-5d1d-4608-b969-5b92fc7bf284.png)

Now, go ahead and make some change in any file in  GitHub repository (e.g. README.MD file) and push the changes to the master branch.

We will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

We have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
Step 3 – Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it

Configure the job/project to copy artifacts over to NFS server.
On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

Provide a private key (content of .pem file that we use to connect to NFS server via SSH/Putty)
Arbitrary name
Hostname – can be private IP address of your NFS server
Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
<img width="1328" alt="Screenshot 2022-07-15 at 4 57 33 PM" src="https://user-images.githubusercontent.com/105562242/179214583-fd24a38b-584f-4501-a053-af2b09434d69.png">

Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"
![image](https://user-images.githubusercontent.com/105562242/179214714-1d798bb2-9c81-4794-9213-8dd3473ea7e2.png)

Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.
  
![image](https://user-images.githubusercontent.com/105562242/179214785-5fff8b2d-be45-4d70-b486-10d82fe6e796.png)
  
Save this configuration and go ahead, change something in README.MD file in GitHub Tooling repository.
  
Here we can see we are getting error message below 
  
<img width="1706" alt="Screenshot 2022-07-13 at 11 55 40 PM" src="https://user-images.githubusercontent.com/105562242/179214940-69d0ce7a-16cc-4a74-8d51-db4ad8faab21.png">

To solve this we will change owership and permisson to /mnt/apps 
  <img width="889" alt="Screenshot 2022-07-14 at 12 02 26 AM" src="https://user-images.githubusercontent.com/105562242/179215043-2976b769-daa3-47b2-be4e-66f9877e1192.png">

After that we can see its working !!
  
<img width="1698" alt="Screenshot 2022-07-14 at 12 03 01 AM" src="https://user-images.githubusercontent.com/105562242/179215120-045df105-aa93-41a4-8280-d9134123c194.png">

To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file
  
<img width="887" alt="Screenshot 2022-07-15 at 5 03 34 PM" src="https://user-images.githubusercontent.com/105562242/179215396-36c0f097-af4a-4cf0-8bca-86df5e9615ae.png">




