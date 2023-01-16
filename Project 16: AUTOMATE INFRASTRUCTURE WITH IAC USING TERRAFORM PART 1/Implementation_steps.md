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

* A new file terraform.tfstate is created as a result of the above command which Terraform uses to keeps itself up to date with the exact state of the infrastructure and terraform.tfstate.lock.info file which Terraform uses to track who is running its code against the infrastructure at any point in time


<img width="334" alt="Screenshot 2023-01-16 at 11 17 56 AM" src="https://user-images.githubusercontent.com/105562242/212607117-f33c2456-7286-4240-aa73-1d27382b19c7.png">

#### STEP 2: Creating Subnet Resources
----------------------------------------------

* According to the architectural design 6 subnets is required: 2 public 2 private for webservers 2 private for data layer
* Creating the 2 public subnets by entering the following codes:
```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1b"
}
```

#### STEP 3: Refactoring The Codes
------------------------------------------
* Inorder to make the work dynamic, hard coded values are removed by introducing variables
* Declaring a variable named region and giving it a default value, and updating the provider section by referring to the declared variable.

```
     variable "region" {
        default = "us-east-1"
    }

    provider "aws" {
        region = var.region
    }
```
```
    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
```
* Fixing multiple resource blocks by introducing some interesting concepts; Loops & Data sources
* Fetching Availability zones from AWS, and replacing the hard coded value in the subnet’s availability_zone section with the use of Data Sources.

```
 # Get list of availability zones
    data "aws_availability_zones" "available" {
        state = "available"
    }
```

* Introducing a count argument in the subnet block to make use of the new data resource:

```
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```
* The count tells terraform to invoke a loop to create 2 subnets. and the data resource will return a list object that contains a list of AZs. But if Terraform is being run with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because the cidr_block still has to be hard coded because the same cidr_block cannot be created twice within the same VPC.
 
* To make the cidr block dynamic a function cidrsubnet() is introduced which accepts 3 parameters.

```
 # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
 ```
* Introuducing length() function, which basically determines the length of a given list and passing it to data.aws_availability_zones.available.names
* But since this returns the value of 3 instead of 2 that is preffered, the variable to store the desired number of public subnets is declared and it is set to the default value.

```
variable "preferred_number_of_public_subnets" {
  default = 2
}

```
* Updating the count argument with a condition of which Terraform checks first if there is a desired number of subnets, Otherwise, it will use the data returned by the lenght function.

```
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}

```
