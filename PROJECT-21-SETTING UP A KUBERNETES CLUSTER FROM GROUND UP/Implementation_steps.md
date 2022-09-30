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

#### SIEP 2: Installing CFSSL And CFSSLJSON
------------------------------------------------------------
The cfssl is an open source tool by Cloudflare used to setup a Public Key Infrastructure(PKI) for generating, signing and bundling TLS certificates

Downloading the binary:

```
$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
  
```
* Making it executable: ```$ chmod +x cfssl cfssljson```
* Moving the file to the bin directory: ```$ sudo mv cfssl cfssljson /usr/local/bin/ ```

<img width="1541" alt="Screenshot 2022-09-08 at 7 47 23 AM" src="https://user-images.githubusercontent.com/105562242/193354078-ec3cbfa5-be3c-47a4-b4c6-1a4e1e1afcbc.png">

#### STEP 3: Configuring The Network Infrastructure
-------------------------------------------------------

* Creating a directory named k8s-cluster-from-ground-up and changing directory:``` $ mkdir k8s-cluster-from-ground-up && cd k8s-cluster-from-ground-up ```
* Creating a VPC and storing its ID as a variable:

```
$ VPC_ID=$(aws ec2 create-vpc \
--cidr-block 172.31.0.0/16 \
--output text --query 'Vpc.VpcId'
)
```
* Tagging the VPC so that it is named:

```
$ NAME=k8s-cluster-from-ground-up

$ aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
```
* Enabling DNS support for the VPC:

```
$ aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-support '{"Value": true}'
```
* Enabling DNS support for hostnames:

```
$ aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-hostnames '{"Value": true}'
```
* Setting the required region:```$ AWS_REGION=us-east-1```
* Configuring DHCP Options Set:

```
$ DHCP_OPTION_SET_ID=$(aws ec2 create-dhcp-options \
  --dhcp-configuration \
    "Key=domain-name,Values=$AWS_REGION.compute.internal" \
    "Key=domain-name-servers,Values=AmazonProvidedDNS" \
  --output text --query 'DhcpOptions.DhcpOptionsId')
```
* Tagging the DHCP Option set:

```
$ aws ec2 create-tags \
  --resources ${DHCP_OPTION_SET_ID} \
  --tags Key=Name,Value=${NAME}
```
* Associating the DHCP Option set with the VPC:
```
$ aws ec2 associate-dhcp-options \
  --dhcp-options-id ${DHCP_OPTION_SET_ID} \
  --vpc-id ${VPC_ID}
```

<img width="1371" alt="Screenshot 2022-09-08 at 7 50 01 AM" src="https://user-images.githubusercontent.com/105562242/193355507-1731af46-2fb9-4a64-9161-7ce8fff13417.png">
