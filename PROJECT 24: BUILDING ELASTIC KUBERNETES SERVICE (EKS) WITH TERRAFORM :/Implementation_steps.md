BUILDING ELASTIC KUBERNETES SERVICE (EKS) WITH TERRAFORM
-----------------------------------------------------------------------
EKS is a managed Kubernetes service that makes it easy for you to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane. In this projetc we will be using Terraform to set it up and futher in the project we will use Helm, which is a package manager for Kubernetes is used to deploy multiple applications.


The following outlines the steps:

STEP 1: Configuring The Terraform Module For EKS
------------------------------------------------------------------------

Creating AWS S3 bucket from a CLI to store the Terraform state:

<img width="1164" alt="Screenshot 2022-09-14 at 8 15 44 PM" src="https://user-images.githubusercontent.com/105562242/209226280-3d7e01ee-2236-48e9-8124-09bef82bd5f0.png">

<img width="1169" alt="Screenshot 2022-09-14 at 8 22 43 PM" src="https://user-images.githubusercontent.com/105562242/209226347-2dc9ca90-7738-470d-b123-f014a81aefbd.png">

<img width="1308" alt="Screenshot 2022-09-14 at 9 23 57 PM" src="https://user-images.githubusercontent.com/105562242/209226523-51e166a6-b771-4677-b3d0-5b9520564a44.png">

<img width="1476" alt="Screenshot 2022-09-14 at 9 53 43 PM" src="https://user-images.githubusercontent.com/105562242/209226784-04c198fd-56a8-45df-9578-49199a422c96.png">


STEP 2: Installing Helm From Script
------------------------------------------------------------------------

* Fetching the script:$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
* Changing the permission of the script:$ chmod 700 get_helm.sh
* Executing the script:$ ./get_helm.sh

<img width="1485" alt="Screenshot 2022-09-14 at 10 26 11 PM" src="https://user-images.githubusercontent.com/105562242/209227056-3d4e4e18-da70-4b27-90ce-f08200cb3f20.png">


STEP 3: Deploying Jenkins With Helm
-------------------------------------------------------------------------
* Adding the Jenkins' repository to helm so it can be easily downloaded and deployed:$ helm repo add jenkins https://charts.jenkins.io
* Updating helm repo:$ helm repo update

<img width="1128" alt="Screenshot 2022-09-14 at 10 27 10 PM" src="https://user-images.githubusercontent.com/105562242/209227416-f4d7ae29-70e8-4001-8f6e-3ce71793f8ab.png">

* Installing the chart:$ helm install myjenkins jenkins/jenkins

<img width="1482" alt="Screenshot 2022-09-14 at 10 31 18 PM" src="https://user-images.githubusercontent.com/105562242/209227495-5d88344d-e7e1-4614-8ccf-22b5d1ca7a80.png">

