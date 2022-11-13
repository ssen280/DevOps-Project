
#### AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

------------------------------------------------------------------

#### INTRODUCTION
------------------------------------------------------------------

This project demonstrates how the AWS infrastructure for 2 websites that was built manually in project 15 is automated with the use of Terraform.

The following outlines the steps taken:


-----------------------------------

#### STEP 0: Setting Up AWS CLI 
-----------------------------------
After creating an IAM user with AdministrativeAccess permissions in AWS and acquiring the access key and secret access key, the following step was taken:


<img width="788" alt="Screenshot 2022-08-23 at 7 52 19 PM" src="https://user-images.githubusercontent.com/105562242/201541901-c8df4869-74cf-491f-ba25-c425c09d9c0c.png">


<img width="840" alt="Screenshot 2022-08-23 at 7 52 29 PM" src="https://user-images.githubusercontent.com/105562242/201541913-4f3425e2-96a1-4663-ae5a-eac44fc29d7a.png">


#### STEP 1: Creating VPC Resource
------------------------------------
* Creating a folder called PBL
* Creating a file in the PBL folder and naming it main.tf
* Entering the following configuration which adds AWS as a provider and a resource to create a VPC in the main.tf file:

<img width="1273" alt="Screenshot 2022-08-23 at 8 13 39 PM" src="https://user-images.githubusercontent.com/105562242/201542019-de895489-eea7-4ee2-8bae-0c52f53bf63a.png">


* Running the following command which downloads the necessary plugins for Terraform to work: terraform init

<img width="1222" alt="Screenshot 2022-08-23 at 8 14 06 PM" src="https://user-images.githubusercontent.com/105562242/201542041-00136623-5c45-41d9-9192-06c3656a9b34.png">

* Inorder to check to see what terraform intends to create before we tell it to go ahead and create the aws_vpc resource the following command is run: terraform plan

![Uploading Screenshot 2022-08-23 at 8.16.45 PM.png…]()

