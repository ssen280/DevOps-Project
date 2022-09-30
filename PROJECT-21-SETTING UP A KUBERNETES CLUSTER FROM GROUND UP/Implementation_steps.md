#### SETTING UP A KUBERNETES CLUSTER FROM GROUND UP (THE HARD WAY)

---------------------------------------------------
#### INTRODUCTION

---------------------------------------------------
In this project, a Kubernetes cluster is manually setup from the scratch without any automated helpers in order to better understand each aspect of spinning up a Kubernetes cluster as each components are manually installed from scratch.

<img width="1123" alt="Screenshot 2022-10-01 at 2 00 12 AM" src="https://user-images.githubusercontent.com/105562242/193351690-ea15b16f-5746-41ae-9801-4c40ab6b9b01.png">

We will create 3 EC2 Instances, and in the end, we will have the following parts of the cluster properly configured:

* One Kubernetes Master
* Two Kubernetes Worker Nodes
* Configured SSL/TLS certificates for Kubernetes components to communicate securely
* Configured Node Network
* Configured Pod Network

The following outlines the steps:

#### STEP 1: Installing Kubectl On The Local Machine(Linux)
---------------------------------------------------------------
* Downloading the binary: $ wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
* Making it executable:$ chmod +x kubectl
* Moving the file to the Bin directory:$ sudo mv kubectl /usr/local/bin/
* Verifying that kubectl version 1.21.0 or higher is installed:$ kubectl version --client

<img width="1539" alt="Screenshot 2022-09-08 at 7 46 11 AM" src="https://user-images.githubusercontent.com/105562242/193353585-11e36b01-0be8-4a1e-8ce6-d5116b783308.png">
