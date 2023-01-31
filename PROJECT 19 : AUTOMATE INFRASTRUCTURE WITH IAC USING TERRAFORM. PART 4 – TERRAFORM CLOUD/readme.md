#### terraform-aws-cloud
-----------------------------------------------
AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology


The sole aim of this project to build the infrastructure in AWS using terraform.

We will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company's desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server's failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

The tooling code is stored in this repository

Always refer to the given diagram


<img width="962" alt="Screenshot 2023-02-01 at 4 23 40 AM" src="https://user-images.githubusercontent.com/105562242/215902247-60cacb04-2698-49eb-ad48-cc2f36b98f35.png">

install graphviz
sudo apt install graphviz

use the command below to generate dependency graph
terraform graph -type=plan | dot -Tpng > graph.png
terraform graph | dot -Tpng > graph.png
Read More abot terrafrom graph
https://www.terraform.io/docs/cli/commands/graph.html

##### Action Plan for project 19

 * Build images using packer
 * confirm the AMIs in the console
 * update terrafrom script with new ami IDs generated from packer build
 * create terraform cloud account and backend
 * run terraform script
 * update ansible script with values from teraform output
 * RDS endpoints for wordpress and tooling
    Database name, password and username for wordpress and tooling
    Access point ID for wordpress and tooling
    Internal load balancee DNS for nginx reverse proxy
    run ansible script
    check the website

Draw back in the scripts
Direct hardcoding of values

Inputting credentials directly in the script
