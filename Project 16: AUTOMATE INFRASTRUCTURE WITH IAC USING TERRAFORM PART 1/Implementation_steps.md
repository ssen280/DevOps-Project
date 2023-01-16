### AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1
----------------------------------------------------------------------

#### INTRODUCTION

----------------------------------
This project demonstrates how the AWS infrastructure for 2 websites that was built manually in project 15 is automated with the use of Terraform.

The following outlines the steps taken:

#### STEP 0: Setting Up AWS CLI And S3 Buckets
-----------------------------
After creating an IAM user with AdministrativeAccess permissions in AWS and acquiring the access key and secret access key, the following step was taken:

* Creating S3 bucket in AWS for storing Terraform state file and naming it eks-terraform-bucket-saikat

<img width="1337" alt="Screenshot 2023-01-16 at 10 28 19 AM" src="https://user-images.githubusercontent.com/105562242/212601889-3cf6d910-c102-4942-9663-f3eabcd26ec6.png">

* We will intall Boto3, aws cli and check cli configuration and we will install terraform 

<img width="877" alt="Screenshot 2023-01-16 at 10 31 38 AM" src="https://user-images.githubusercontent.com/105562242/212602219-8dd04874-0fdd-403a-9484-80013845f79a.png">

#### STEP 1: Creating VPC Resource
-------------------------------

* Creating a project folder.
* Creating a file in the folder and naming it main.tf
* Entering the following configuration which adds AWS as a provider and a resource to create a VPC in the main.tf file:

```
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}

```
* Running the following command which downloads the necessary plugins for Terraform to work: terraform init

<img width="1065" alt="Screenshot 2023-01-16 at 9 16 16 AM" src="https://user-images.githubusercontent.com/105562242/212606618-31b319c0-be2e-41e6-a546-44de62e1b1fd.png">

* Inorder to check to see what terraform intends to create before we tell it to go ahead and create the aws_vpc resource the following command is run: terraform plan

<img width="1152" alt="Screenshot 2023-01-16 at 9 17 48 AM" src="https://user-images.githubusercontent.com/105562242/212606725-d54f70d8-cb0f-4224-8a2b-1cb31c6e0684.png">

* Then executing the plan: terraform apply

<img width="1158" alt="Screenshot 2023-01-16 at 9 20 44 AM" src="https://user-images.githubusercontent.com/105562242/212606957-61f5811a-9d7d-4c5b-a9d0-6f7b9e8ebf0b.png">

<img width="334" alt="Screenshot 2023-01-16 at 11 17 56 AM" src="https://user-images.githubusercontent.com/105562242/212607117-f33c2456-7286-4240-aa73-1d27382b19c7.png">

#### STEP 2: Creating Subnet Resources
----------------------------------------------
