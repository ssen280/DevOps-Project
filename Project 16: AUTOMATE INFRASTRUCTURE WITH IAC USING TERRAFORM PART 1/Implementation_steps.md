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
