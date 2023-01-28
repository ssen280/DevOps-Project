#### Automate Infrastructure With IaC using Terraform. Part 4 - Terraform Cloud.
------------------------------------------------------------------------------------

In this project we will be using Packer to build our images, and Ansible to configure the infrastructure, so for that we are going to make few changes to our our existing repository from Project 18.

The files that would be addedd are:

* AMI: for building packer images
* Ansible: for Ansible scripts to configure the infrastructure

##### Action Plan for project 19

* Build images using packer
* confirm the AMIs in the console
* update terrafrom script with new ami IDs generated from packer build
* create terraform cloud account and backend
* run terraform script
* update ansible script with values from teraform output
  * RDS endpoints for wordpress and tooling
  * 
