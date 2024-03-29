
In this project, we will understand and get hands on experience around the entire concept around CI/CD from applications perspective. To fully gain real expertise around this idea, it is best to see it in action across different programming languages and from the platform perspective too. From the application perspective, we will be focusing on PHP here

#### What is Continuous Integration?
In software engineering, Continuous Integration (CI) is a practice of merging all developers’ working copies to a shared mainline (e.g., Git Repository or some other version control system) several times per day. Frequent merges reduce chances of any conflicts in code and allow to run tests more often to avoid massive rework if something goes wrong. This principle can be formulated as Commit early, push often.

The general idea behind multiple commits is to avoid what is generally considered as Merge Hell or Integration hell. When a new developer joins a new project, he or she must create a copy of the main codebase by starting a new feature branch from the mainline to develop his own features (in some organization or team, this could be called a develop, main or master branch). If there are tens of developers working on the same project, they will all have their own branches created from mainline at different points in time. Once they make a copy of the repository it starts drifting away from the mainline with every new merge of other developers’ codes. If this lingers on for a very long time without reconciling the code, then this will cause a lot of code conflict or Merge Hell, as rightly said. Imagine such a hell from tens of developers or worse, hundreds. So, the best thing to do, is to continuously commit & push your code to the mainline. As many times as tens times per day. With this practice, you can avoid Merge Hell or Integration hell.

CI concept is not only about committing your code. There is a general workflow, let us start it…

1. **Run tests locally:** Before developers commit their code to a central repository, it is recommended to test the code locally. So, Test-Driven Development (TDD) approach is commonly used in combination with CI. Developers write tests for their code called unit-tests, and before they commit their work, they run their tests locally. This practice helps a team to avoid having one developer’s work-in-progress code from breaking other developers’ copy of the codebase.

2. **Compile code in CI:** After testing codes locally, developers commit and push their work to a central repository. Rather than building the code into an executable locally, a dedicated CI server picks up the code and runs the build there. In this project we will use, already familiar to you, Jenkins as our CI server. Build happens either periodically – by polling the repository at some configured schedule, or after every commit. Having a CI server where builds run is a good practice for a team, as everyone has visibility into each commit and its corresponding builds.

3. **Run further tests in CI:** Even though tests have been run locally by developers, it is important to run the unit-tests on the CI server as well. But, rather than focusing solely on unit-tests, there are other kinds of tests and code analysis that can be run using CI server. These are extremely critical to determining the overall quality of code being developed, how it interacts with other developers’ work, and how vulnerable it is to attacks. A CI server can use different tools for Static Code Analysis, Code Coverage Analysis, Code smells Analysis, and Compliance Analysis. In addition, it can run other types of tests such as Integration and Penetration tests. Other tasks performed by a CI server include production of code documentation from the source code and facilitate manual quality assurance (QA) testing processes.

4. **Deploy an artifact from CI:** At this stage, the difference between CI and CD is spelt out. As you now know, CI is Continuous Integration, which is everything we have been discussing so far. CD on the other hand is Continuous Delivery which ensures that software checked into the mainline is always ready to be deployed to users. The deployment here is manually triggered after certain QA tasks are passed successfully. There is another CD known as Continuous Deployment which is also about deploying the software to the users, but rather than manual, it makes the entire process fully automated. Thus, Continuous Deployment is just one step ahead in automation than Continuous Delivery.

Continuous Integration in The Real World
To emphasize a typical CI Pipeline further, let us explore the diagram below a little deeper.


<img width="903" alt="Screenshot 2022-08-21 at 11 28 03 PM" src="https://user-images.githubusercontent.com/105562242/185804537-6481f631-b90a-41e3-88ff-a534cf09edaa.png">

**Version Control:** This is the stage where developers’ code gets committed and pushed after they have tested their work locally.
Build: Depending on the type of language or technology used, we may need to build the codes into binary executable files (in case of compiled languages) or just package the codes together with all necessary dependencies into a deployable package (in case of interpreted languages).

**Unit Test:** Unit tests that have been developed by the developers are tested. Depending on how the CI job is configured, the entire pipeline may fail if part of the tests fails, and developers will have to fix this failure before starting the pipeline again. A Job by the way, is a phase in the pipeline. Unit Test is a phase, therefore it can be considered a job on its own.

**Deploy:** Once the tests are passed, the next phase is to deploy the compiled or packaged code into an artifact repository. This is where all the various versions of code including the latest will be stored. The CI tool will have to pick up the code from this location to proceed with the remaining parts of the pipeline.

**Auto Test:** Apart from Unit testing, there are many other kinds of tests that are required to analyse the quality of code and determine how vulnerable the software will be to external or internal attacks. These tests must be automated, and there can be multiple environments created to fulfil different test requirements. For example, a server dedicated for Integration Testing will have the code deployed there to conduct integration tests. Once that passes, there can be other sub-layers in the testing phase in which the code will be deployed to, so as to conduct further tests. Such are User Acceptance Testing (UAT), and another can be Penetration Testing. These servers will be named according to what they have been designed to do in those environments. A UAT server is generally be used for UAT, SIT server is for Systems Integration Testing, PEN Server is for Penetration Testing and they can be named whatever the naming style or convention in which the team is used. An environment does not necessarily have to reside on one single server. In most cases it might be a stack as you have defined in your Ansible Inventory. All the servers in the inventory/dev are considered as Dev Environment. The same goes for inventory/stage (Staging Environment) inventory/preprod (Pre-production environment), inventory/prod (Production environment), etc. So, it is all down to naming convention as agreed and used company or team wide.

**Deploy to production:** Once all the tests have been conducted and either the release manager or whoever has the authority to authorize the release to the production server is happy, he gives green light to hit the deploy button to ship the release to production environment. This is an Ideal Continuous Delivery Pipeline. If the entire pipeline was automated and no human is required to manually give the Go decision, then this would be considered as Continuous Deployment. Because the cycle will be repeated, and every time there is a code commit and push, it causes the pipeline to trigger, and the loop continues over and over again.

**Measure and Validate:** This is where live users are interacting with the application and feedback is being collected for further improvements and bug fixes. There are many metrics that must be determined and observed here. We will quickly go through 13 metrics that MUST be considered.

**Common Best Practices of CI/CD
Before we move on to observability metrics – let us list down the principles that define a reliable and robust CI/CD pipeline:**

- Maintain a code repository
- Automate build process
- Make builds self-tested
- Everyone commits to the baseline every day
- Every commit to baseline should be built
- Every bug-fix commit should come with a test case
- Keep the build fast
- Test in a clone of production environment
- Make it easy to get the latest deliverables
- Everyone can see the results of the latest build
- Automate deployment (if you are confident enough in your CI/CD pipeline and willing to go for a fully automated Continuous Deployment)

**13 DevOps success metrics**

1. **Deployment frequency:** Tracking how often you do deployments is a good DevOps metric. Ultimately, the goal is to do more smaller deployments as often as possible. Reducing the size of deployments makes it easier to test and release. I would suggest counting both production and non-production deployments separately. How often you deploy to QA or pre-production environments is also important. You need to deploy early and often in QA to ensure enough time for testing.

2. **Lead time:** If the goal is to ship code quickly, this is a key DevOps metric. I would define lead time as the amount of time that occurs between starting on a work item until it is deployed. This helps you know that if you started on a new work item today, how long would it take on average until it gets to production.

3. **Customer tickets:** The best and worst indicator of application problems is customer support tickets and feedback. The last thing you want is your users reporting bugs or having problems with your software. Because of this, customer tickets also serve as a good indicator of application quality and performance problems.

4. **Percentage of passed automated tests:** To increase velocity, it is highly recommended that the development team makes extensive usage of unit and functional testing. Since DevOps relies heavily on automation, tracking how well automated tests work is a good DevOps metrics. It is good to know how often code changes break tests.

5. **Defect escape rate:** Do you know how many software defects are being found in production versus QA? If you want to ship code fast, you need to have confidence that you can find software defects before they get to production. Defect escape rate is a great DevOps metric to track how often those defects make it to production.

6. **Availability:** The last thing we ever want is for our application to be down. Depending on the type of application and how we deploy it, we may have a little downtime as part of scheduled maintenance. It is highly recommended to track this metric and all unplanned outages. Most software companies build status pages to track this. Such as this Google Products Status Page

7. **Service level agreements:** Most companies have some service level agreement (SLA) that they promise to the customers. It is also important to track compliance with SLAs. Even if there are no formally stated SLAs, there probably are application non-functional requirements or expectations to be met.

8. **Failed deployments:** We all hope this never happens, but how often do our deployments cause an outage or major issues for the users? Reversing a failed deployment is something we never want to do, but it is something you should always plan for. If you have issues with failed deployments, be sure to track this metric over time. This could also be seen as tracking *Mean Time To Failure (MTTF).

9. **Error rates:** Tracking error rates within the application is super important. Not only they serve as an indicator of quality problems, but also ongoing performance and uptime related issues. In software development, errors are also known as exceptions, and proper exception handling is critical. If they are not handled nicely, we can figure it out while monitoring the rate of errors.

- Bugs – Identify new exceptions being thrown in the code after a deployment
- Production issues – Capture issues with database connections, query timeouts, and other related issues

Presenting error rate metrics like this simply gives greater insights into where to focus attention.

10. **Application usage & traffic:** After a deployment, we want to see if the number of transactions or users accessing our system looks normal. If we suddenly have no traffic or a giant spike in traffic, something could be wrong. An attacker may be routing traffic elsewhere, or initiating a DDOS attack

11. **Application performance:** Before we even perform a deployment, we should configure monitoring tools like Retrace, DataDog, New Relic, or AppDynamics to look for performance problems, hidden errors, and other issues. During and after the deployment, we should also look for any changes in overall application performance and establish some benchmarks to know when things deviate from the norm.

It might be common after a deployment to see major changes in the usage of specific SQL queries, web service or HTTP calls, and other application dependencies. These monitoring tools can provide valuable visualizations like this one below that helps make it easy to spot problems.



12. **Mean time to detection (MTTD):** When problems happen, it is important that we identify them quickly. The last thing we want is to have a major partial or complete system outage and not know about it. Having robust application monitoring and good observability tools in place will help us detect issues quickly. Once they are detected, we also must fix them quickly!

13. **Mean time to recovery (MTTR):** This metric helps us track how long it takes to recover from failures. A key metric for the business is keeping failures to a minimum and being able to recover from them quickly. It is typically measured in hours and may refer to business hours, not calendar hours.

These are the major metrics that any DevOps team should track and monitor to understand how well CI/CD process is established and how it helps to deliver quality application to the users.

------------------------------------------------------------------------------------------------------------------------------------------
#### So We are going to do as below in this project :

##### In this project, the concept of CI/CD is implemented whereby php application from github are pushed to Jenkins to run a multi-branch pipeline job(build job is run on each branches of a repository simultaneously) which is better viewed from Blue Ocean plugin. This is done in order to achieve continuous integration of codes from different developers. After which the artifacts from the build job is packaged and pushed to sonarqube server for testing before it is deployed to artifactory from which ansible job is triggered to deploy the application to production environment.

#### 1. Setting Up Servers:

##### I launched 3 EC2 Instances, one for Jenkins server, another for MySQL database(RedHat) and another is used for SonarQube Server

#### 2. Configuring Ansible For Jenkins Deployment:

##### In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.To do this,

##### Navigate to Jenkins URL

##### Install & Open Blue Ocean Jenkins Plugin

##### Create a new pipeline


<img width="1052" alt="Screenshot 2022-08-22 at 8 59 08 AM" src="https://user-images.githubusercontent.com/105562242/185832887-a0d82989-33b6-48ec-bbfb-bc62c64f75d6.png">

##### Select GitHub
<img width="1084" alt="Screenshot 2022-08-22 at 8 59 34 AM" src="https://user-images.githubusercontent.com/105562242/185832918-c03a81b1-33b3-4fb6-b9ae-6c5d8e6a03f5.png">

##### Connect Jenkins with GitHub

<img width="1085" alt="Screenshot 2022-08-22 at 9 03 25 AM" src="https://user-images.githubusercontent.com/105562242/185833299-0015fbc9-5438-4ce2-a1e7-6326a250ccee.png">


<img width="1078" alt="Screenshot 2022-08-22 at 9 04 07 AM" src="https://user-images.githubusercontent.com/105562242/185833358-c478d507-d99a-4938-998a-3d4c1f70073b.png">
<img width="1027" alt="Screenshot 2022-08-22 at 9 04 25 AM" src="https://user-images.githubusercontent.com/105562242/185833387-2cb1c47f-7731-4dac-9d41-cd821b070ac8.png">

##### Copy Access Token
<img width="1066" alt="Screenshot 2022-08-22 at 9 05 09 AM" src="https://user-images.githubusercontent.com/105562242/185833473-fcdaa093-c214-473e-8c3e-e99b0d8a9dd3.png">

##### Paste the token and connect

<img width="1061" alt="Screenshot 2022-08-22 at 9 05 36 AM" src="https://user-images.githubusercontent.com/105562242/185833529-4b78cd8d-18be-4f76-a08f-faee5ead47ac.png">

##### Create a new Pipeline, choose the new repo and click on complete. our newly created pipeline. It will take name of selected GitHub repository.
<img width="992" alt="Screenshot 2022-08-22 at 9 15 36 AM" src="https://user-images.githubusercontent.com/105562242/185834453-9a9bce0c-6d1d-4ec4-8614-a96c09daf53a.png">

##### Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.

<img width="237" alt="Screenshot 2022-08-22 at 9 36 55 AM" src="https://user-images.githubusercontent.com/105562242/185837611-d54f2422-3c12-47ae-a2e5-952c2dc80a97.png">

##### Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```

##### Now go back into the Ansible pipeline in Jenkins, and select configure

##### Now go back into the Ansible pipeline in Jenkins, and select configure
##### Back to the pipeline again, this time click "Build now"
##### This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

##### To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.Select the project. Click on the play button against the branch

##### Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

##### Let us see this in action.

##### Create a new git branch and name it feature/<repo name>. Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.

```
   pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```
##### To make our new branch show up in Jenkins, we need to tell Jenkins to scan the repository. Click on the "Administration" button
##### Navigate to the Ansible project and click on "Scan repository now"
##### Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.
##### In Blue Ocean, we can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

<img width="1721" alt="Screenshot 2022-08-12 at 1 05 13 AM" src="https://user-images.githubusercontent.com/105562242/185844143-5db7ceee-1299-491c-bcb7-c19e013ccd1c.png">

<img width="1687" alt="Screenshot 2022-08-22 at 10 48 07 AM" src="https://user-images.githubusercontent.com/105562242/185844587-8a82da71-3ab0-4553-9bd9-d16931d7ffd2.png">

 <img width="1713" alt="Screenshot 2022-08-12 at 1 05 32 AM" src="https://user-images.githubusercontent.com/105562242/185844749-3dcf6c46-27c8-4a47-ba4e-2b7f99640981.png">

 <img width="1728" alt="Screenshot 2022-08-12 at 1 06 19 AM" src="https://user-images.githubusercontent.com/105562242/185844775-e4c5127e-d1ed-4c27-be7d-9f97dedb3816.png">

    ##### We will create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
   1. Package 
   2. Deploy 
   3. Clean up

##### - Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
##### - Eventually, your main branch should have a successful pipeline like this in blue ocean
  
```
  pipeline {
   agent any

 stages {
   stage("Initial cleanup"){
     steps {
       dir("${WORKSPACE}"){
         deleteDir()
       }
     }
   }
   
    stage('Build') {
      steps {
        script {
         sh 'echo "Building Stage"'
       }
     }
   }

   stage('Test') {
     steps {
       script {
         sh 'echo "Testing Stage"'
       }
     }
   }
   stage('Package') {
     steps {
       script {
         sh 'echo "Packaging Stage"'
       }
     }
   }
   stage('Deploy') {
     steps {
       script {
         sh 'echo "Deploying Stage"'
       }
     }
   }
   stage('Clean up') {
     steps {
       cleanWs()
       }
     }
  
   }
 }
```
<img width="1718" alt="Screenshot 2022-08-12 at 5 23 49 AM" src="https://user-images.githubusercontent.com/105562242/185845977-44e3eb86-3817-40de-aa52-6dd7f216da41.png">

#### Running Ansible playbook from Jenkins

##### Installing Ansible plugin from manage plugins on Jenkins to run ansible commands
##### Clearing the codes in the Jenkinsfile to start from the scratch
##### For ansible to be able to locate the roles in my playbook, the ansible.cfg is copied to '/deploy' folder and then exported from the Jenkinsfile code
    
##### The structure of the deployment folder now 
    
<img width="822" alt="Screenshot 2022-08-22 at 11 24 41 AM" src="https://user-images.githubusercontent.com/105562242/185848951-5c1316f8-4804-4de2-bbca-5213853a8333.png">

##### Ansible config file details :
    
```
[defaults]
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true


[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes
```
 ##### We will use jenkins pipeline syntax Ansible tool to generate syntax for executing playbook.
 
 ##### We have to populate below fileds. 
 - Playbook file path in workspace ( playbook: 'playbooks/site.yaml')
 - Inventory file path in workspace ( inventory: 'inventory/dev.yaml')
 - We have select ssh communication so that Jenkin can communication with server through ssh cert. same cert which we use to access servers. and we have to click on 'generate pipeline script button'
 - After pupulating above mentioned fileds and generating script. we have to use that script in our jenkinsfile. which is look like as below 
    
 ```
 ansiblePlaybook become: true, colorized: true, credentialsId: 'ssh-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/dev.yaml', playbook: 'playbooks/site.yaml'
 ```

<img width="1266" alt="Screenshot 2022-08-22 at 11 38 47 AM" src="https://user-images.githubusercontent.com/105562242/185850826-1893c355-1924-4612-9fba-615dc5f5de18.png">

##### Introducing parameterization which enables us to input the appropriate values for the inventory file we want playbook to run against:
```
  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
```
<img width="1597" alt="Screenshot 2022-08-22 at 7 26 45 PM" src="https://user-images.githubusercontent.com/105562242/185938680-2ed98c6c-7fa5-40d6-bb34-f083bab274fb.png">

##### Now in the Ansible execution section, We will remove the hardcoded inventory/dev and replace with `${inventory} so that we can get option to choose our desire dev/test/pod environment. 
    
##### Final ansible file looks like as below 
    
```
pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }

  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }

  stages{
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

      stage('Checkout SCM') {
         steps{
            git branch: 'main', url: 'https://github.com/ssen280/PROJECT-14-ansible-configuration-management.git'
         }
       }

      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
     }

      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, colorized: true, credentialsId: 'ssh-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yaml'
         }
      }

      stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }

}
```
#### Setting Up The Artifactory Server:
-------------------------------------------------
##### Since the goal here is to deploy applications directory from Artifactory rather than git, the following step is taken:

##### - We will create one account on artifactory server.
##### - We will the repository where the artifacts will be uploaded to from the Jenkins server
<img width="1718" alt="Screenshot 2022-08-22 at 7 44 09 PM" src="https://user-images.githubusercontent.com/105562242/185942729-45621993-b7ee-4401-9a02-2f337163c7b6.png">

#####  Integrating Artifactory Repository With Jenkins:
--------------------------------------------------
##### - Installing plot plugin to display tests reports and code coverage and Artifactory plugins to easily upload code artifacts into an Artifactory server
##### - Configuring Artifactory in Jenkins on ‘configure systems’
<img width="1243" alt="Screenshot 2022-08-22 at 10 45 23 PM" src="https://user-images.githubusercontent.com/105562242/185980456-895cf202-6ddc-4f69-998d-474664d1e740.png">

##### - Forking the repository into my Github account: https://github.com/darey-devops/php-todo.git
##### - On database server, installing mysql: $ sudo yum install mysql-server
##### - Creating a database and a remote user:
<img width="1120" alt="Screenshot 2022-08-23 at 10 21 18 AM" src="https://user-images.githubusercontent.com/105562242/186072550-ee921441-e118-47f8-8eb5-4c9535b3186e.png">
```
Create database homestead;
CREATE USER 'homestead'@'<Jenkins-ip-address>' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'<Jenkins-ip-address>';
```
##### - Configuring the bind_address in the mysqld.cnf file:
<img width="1121" alt="Screenshot 2022-08-23 at 10 29 14 AM" src="https://user-images.githubusercontent.com/105562242/186073510-1b20378f-6458-48f9-8794-6f1e9ba7ee8a.png">
##### - Opening port 3306 in the database security group
<img width="1364" alt="Screenshot 2022-08-23 at 10 30 03 AM" src="https://user-images.githubusercontent.com/105562242/186073630-1c215463-0fd4-4c39-ad5c-ca3086c3e9b0.png">
##### - On the Jenkins server, installing PHP, its dependencies. We have to make sure that we are installing php7.4, latest php version (v8) will give error. We will use below link to do the installtion
    
https://www.digitalocean.com/community/tutorials/how-to-install-php-7-4-and-set-up-a-local-development-environment-on-ubuntu-18-04

<img width="1039" alt="Screenshot 2022-08-23 at 11 11 10 AM" src="https://user-images.githubusercontent.com/105562242/186078881-1a6fb2e0-16ea-49be-973e-0aaca1fcefe0.png">

##### - Updating the .env.sample with database server ip address
<img width="641" alt="Screenshot 2022-08-23 at 11 12 32 AM" src="https://user-images.githubusercontent.com/105562242/186079054-6fe73e6a-e036-4754-8317-ad0439416bbc.png">

##### - Creating Jenkinsfile for the php-todo repository on the main branch
```
pipeline {
    agent any

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

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```
##### The first stage performs a clean-up which always deletes the previous workspace before running a new one

##### The second stage connects to the php-todo repository

##### The third stage performs a shell scripting which renames .env.sample file to .env, then composer is used by PHP to install all the dependent libraries used by the application and then php artisan uses the .env to setup the required database objects

##### Pushing the changes to the main branch: git push
##### Creating php-todo pipeline job from the Blue Ocean UI
##### Running the pipeline job
 <img width="1685" alt="Screenshot 2022-08-23 at 11 22 34 AM" src="https://user-images.githubusercontent.com/105562242/186080463-443aba7f-4206-4862-ad8a-e2d42659e471.png">

##### - After a successful run of the pipeline job, confirming on the database server by running SHOW TABLES command to see the tables being created
 
 <img width="1143" alt="Screenshot 2022-08-23 at 11 25 47 AM" src="https://user-images.githubusercontent.com/105562242/186080889-a67efe74-df8a-4e95-bb96-cd0d8b5ef899.png">

##### - Structuring The Jenkinsfile:
------------------------------------------------------------------------------
##### - Including unit test stage in the Jenkinsfile:
```
stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      } 
    }
```
##### - Adding Code Quality stage with phploc tool and will save the output in build/logs/phploc.csv:
```
stage('Code Analysis') {
      steps {
        sh 'phploc app/ --log-csv build/logs/phploc.csv'

      }
    }
```
##### - Adding the plot code coverage report stage:
```
stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }
```
##### - Adding the package artifacts stage which archives the application code
```
    stage('Package Artifact') {
      steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
      }
    }
```
##### - Uploading the artifacts to the Artifactory repository in this stage:
```
    stage('Upload Artifact to Artifactory') {
      steps {
        script { 
                def server = Artifactory.server 'artifactory'                 
                def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "artifacts-todo-php",
                       "props": "type=zip;status=ready"

                      }
                    ]
                }""" 

              server.upload spec: uploadSpec
        }
      }
    }
 ```
##### - Deploying the application to the dev environment by launching Ansible pipeline job(ansible-config-mgt)
```
    stage ('Deploy to Dev Environment') {
      steps {
        build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
      }
    }
```
#### Setting Up The SonarQube Server:
-----------------------------------------
##### - We will follow steps from below link to install SonarQube on server

https://www.vultr.com/docs/install-sonarqube-on-ubuntu-20-04-lts/


##### - Opening TCP port 9000 on the security group
<img width="1020" alt="Screenshot 2022-08-23 at 11 50 10 AM" src="https://user-images.githubusercontent.com/105562242/186084922-91498a9a-5d1d-41fb-aafa-0a3893a4f829.png">

##### - Accessing SonarQube through the browser by entering the SonarQube server’s IP address followed by port 9000: http://<server_IP_adress>:9000

#### Configuring Jenkins For SonarQube Quality Gate
----------------------------------------------------------------
##### - Generating authentication token in the SonarQube server by navigating from 'My Account' to 'security'
##### - Configuring Quality Gate Jenkins Webhook in SonarQube by navigating from 'Administration' to 'Configuration' to 'webhook' and 'create' and then specifying the URL as this: http://<Jenkins ip address>/sonarqube-webhook/
##### - Installing SonarScanner plugin in jenkins
##### - Navigating to 'Configure System' in Jenkins to add SonarQube server details with the generated token
##### - Setting the SonarQube scanner by navigating from 'manage jenkins' to 'Global Tool Configuration'

  <img width="1249" alt="Screenshot 2022-08-23 at 11 59 43 AM" src="https://user-images.githubusercontent.com/105562242/186086604-5da75901-fe93-4564-8749-6df42d3063e5.png">

##### - Updating the Jenkinsfile to include SonarQube Scanning and Quality Gate
```

stage('SonarQube Quality Gate') {
      environment {
          scannerHome = tool 'SonarQubeScanner'
      }
        steps {
          withSonarQubeEnv('sonarqube') {
              sh "${scannerHome}/bin/sonar-scanner"
          }

        }
    }
    
```
##### - Configuring the sonar-scanner.properties file in which SonarQube will require to function during pipeline execution: $ cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/
    
##### - Editing and entering the following configuration: $ sudo vi sonar-scanner.properties
    
```
sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
```
#### - Running The Pipeline Job
---------------------------------------
    
##### - Our final jenkinsfile in php-todo repo looks like as below 
```
pipeline {
    agent any

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

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }

      stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      } 
  }
  stage('Code Analysis') {
  steps {
        sh 'phploc app/ --log-csv build/logs/phploc.csv'

  }
}

stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }

    stage('SonarQube Quality Gate') {
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }

        }
    }

    stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }

    }
    stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifectory-project14'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "Saikat/php-todo",
                       "props": "type=zip;status=ready"
                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }
  
        }

  stage ('Deploy to Dev Environment') {
    steps {
    build job: 'PROJECT-14-ansible-configuration-management/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }


}
}
```
##### - Here we can see we are getting error. 
<img width="1650" alt="Screenshot 2022-08-23 at 12 06 47 PM" src="https://user-images.githubusercontent.com/105562242/186087776-2d2392b6-4b2f-4ee6-875c-d17b9ad9d88c.png">
<img width="1316" alt="Screenshot 2022-08-23 at 12 07 41 PM" src="https://user-images.githubusercontent.com/105562242/186087923-453be404-fd86-4230-b899-283f2372f815.png">

##### - So we will install node.js on jenkin server to avoide this error 
    
<img width="614" alt="Screenshot 2022-08-23 at 12 14 08 PM" src="https://user-images.githubusercontent.com/105562242/186089112-afb387aa-4e44-41fd-b40b-d62275db8d92.png">
##### - Now we can see error is gone and pipeline jobs run successfully 
  <img width="1693" alt="Screenshot 2022-08-23 at 12 15 28 PM" src="https://user-images.githubusercontent.com/105562242/186089346-743d00f0-7b57-44fb-b680-e503f5d57413.png">

##### - Results from the SonarQube Server

<img width="1612" alt="Screenshot 2022-08-18 at 11 27 35 PM" src="https://user-images.githubusercontent.com/105562242/186089873-41642950-6edb-4fb0-b8c2-cba1b5f356e2.png">
##### - From the result it shows that there are bugs, and there is 0.0% code coverage(code coverage is a percentage of unit tests added by developers to test functions and objects in the code) and there is 6 hours’ worth of technical debt, code smells and security issues in the code. And therefore as DevOps Engineer working on the pipeline, we must ensure that the quality gate step causes the pipeline to fail if the conditions for quality are not met.

##### - To ensure that only pipeline job that is run on either 'main' or 'develop' or 'hotfix' or 'release' branch gets to make it to the deploy stage, the Jenkinsfile is updated and a timeout step is also added to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable.:

```
 stage('SonarQube Quality Gate') {
      when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dsonar.projectKey=php-todo"
            }
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
```
##### - Here we can see our artiface upload to JFrog during Package Artifact and Upload Artifact to Artifactory pipeling jobs
    

<img width="1701" alt="Screenshot 2022-08-18 at 11 28 35 PM" src="https://user-images.githubusercontent.com/105562242/186091090-2aa26ed7-6e44-47a4-a47e-208d61c37ded.png">



 
