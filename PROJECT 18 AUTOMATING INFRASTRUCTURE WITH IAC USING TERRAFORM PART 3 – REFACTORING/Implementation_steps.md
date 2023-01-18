
#### AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 3 – REFACTORING
------------------------------------------------------------------------------------
#### INTRODUCTION
------------------------------------------------------------------------------------
In continuation to Project 17, the entire code is refactored inorder to simplify the code using a Terraform tool called Module.

The following outlines detailed step taken to achieve this:

#### STEP 1: Configuring A Backend On The S3 Bucket
--------------------------------------------------------------------------------------
By default the Terraform state is stored locally, to store it remotely on AWS using S3 bucket as the backend and also making use of DynamoDB as the State Locking the following setup is done:

* Creating a file called Backend.tf and entering the following code:

##### backend.tf
```
resource "aws_s3_bucket" "terraform_state" {
  bucket = "project19-bucket-saikat"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}

```
* Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run terraform apply to provision resources.

<img width="1065" alt="Screenshot 2023-01-18 at 7 58 59 AM" src="https://user-images.githubusercontent.com/105562242/213070082-a9ab156f-6179-482e-bd18-a1568db95df1.png">

<img width="1064" alt="Screenshot 2023-01-18 at 7 59 13 AM" src="https://user-images.githubusercontent.com/105562242/213070115-d7ee91e4-82c0-47db-898e-f014319b73bc.png">

* Configure S3 Backend. 

```
terraform {
  backend "s3" {
    bucket         = "project19-bucket-saikat"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

```
* Now its time to re-initialize the backend. Run terraform init and confirm you are happy to change the backend by typing yes
* Verify the changes

<img width="1069" alt="Screenshot 2023-01-18 at 8 04 40 AM" src="https://user-images.githubusercontent.com/105562242/213070763-5c89bf9a-9d5a-4c32-9be3-09b322e7c235.png">


<img width="1398" alt="Screenshot 2023-01-18 at 7 57 38 AM" src="https://user-images.githubusercontent.com/105562242/213070421-e28949f7-9b4d-4d23-8f63-19b16af7005e.png">

<img width="1383" alt="Screenshot 2023-01-18 at 7 58 08 AM" src="https://user-images.githubusercontent.com/105562242/213070443-f21f6fc5-07a2-47be-a509-3f4a373f4c7d.png">

<img width="1404" alt="Screenshot 2023-01-18 at 8 21 10 AM" src="https://user-images.githubusercontent.com/105562242/213070593-bba1a694-0f32-497d-9304-7b85505995c4.png">

* Add Terraform Output. Before to run terraform apply let us add an output so that the S3 bucket Amazon Resource Names ARN and DynamoDB table name can be displayed.

##### output.tf

```
output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}
output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}

```
<img width="1060" alt="Screenshot 2023-01-18 at 8 12 51 AM" src="https://user-images.githubusercontent.com/105562242/213071221-d8dd611d-3e8f-4542-aa6b-6e276146102d.png">


#### STEP 2: Refactoring The Codes Using Module
----------------------------------------------------------
* Creating a folder called modules
* Creating the following folders inside the modules folder to combine resources of the similar type: ALB, VPC, Autoscaling, Security, EFS, RDS, Compute
* Creating the following files for each of the folders: main.tf, variables.tf and output.tf

##### pbl folder structure

<img width="261" alt="Screenshot 2023-01-18 at 8 31 54 AM" src="https://user-images.githubusercontent.com/105562242/213072238-4c940449-885c-4e7d-961f-2e62d156737d.png">

* Refactoring the code for VPC folder:

##### main.tf
```
# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support
  enable_dns_hostnames           = var.enable_dns_hostnames
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_dns_support


  tags = merge(
    var.tags,
    {
      Name = format("%s-VPC", var.name)
    },
  )
}

# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Create public subnets
resource "aws_subnet" "public" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnets[count.index]
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]



  tags = merge(
    var.tags,
    {
      Name = format("%s-PublicSubnet-%s", var.name, count.index)
    },
  )

}

# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.private_subnets[count.index]
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

  tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}

```
<img width="1008" alt="Screenshot 2023-01-18 at 8 36 21 AM" src="https://user-images.githubusercontent.com/105562242/213072981-a189996a-f849-45b1-b4c9-01b0d43281dd.png">

* Refactoring the code for ALB folder:

##### variables.tf

```
# The security froup for external loadbalancer
variable "public-sg" {
  description = "Security group for external load balancer"
}


# The public subnet froup for external loadbalancer
variable "public-sbn-1" {
  description = "Public subnets to deploy external ALB"
}
variable "public-sbn-2" {
  description = "Public subnets to deploy external  ALB"
}


variable "vpc_id" {
  type        = string
  description = "The vpc ID"
}


variable "private-sg" {
  description = "Security group for Internal Load Balance"
}

variable "private-sbn-1" {
  description = "Private subnets to deploy Internal ALB"
}
variable "private-sbn-2" {
  description = "Private subnets to deploy Internal ALB"
}

variable "ip_address_type" {
  type        = string
  description = "IP address for the ALB"

}

variable "load_balancer_type" {
  type        = string
  description = "te type of Load Balancer"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}


variable "name" {
    type = string
    description = "name of the loadbalancer"
  
}

```
<img width="1010" alt="Screenshot 2023-01-18 at 8 38 23 AM" src="https://user-images.githubusercontent.com/105562242/213073642-e60116c8-1a58-4be8-8e9b-602056d968e5.png">

##### outputs.tf
<img width="860" alt="Screenshot 2023-01-18 at 8 42 58 AM" src="https://user-images.githubusercontent.com/105562242/213074530-ca2a110b-1cc2-44bf-9b67-bc4b34662803.png">

* Refactoring the code for Autoscaling folder:

```
variable "ami-web" {
  type        = string
  description = "ami for webservers"
}

variable "instance_profile" {
  type        = string
  description = "Instance profile for launch template"
}


variable "keypair" {
  type        = string
  description = "Keypair for instances"
}

variable "ami-bastion" {
  type        = string
  description = "ami for bastion"
}

variable "web-sg" {
  type = list
  description = "security group for webservers"
}

variable "bastion-sg" {
  type = list
  description = "security group for bastion"
}

variable "nginx-sg" {
  type = list
  description = "security group for nginx"
}

variable "private_subnets" {
  type = list
  description = "first private subnets for internal ALB"
}


variable "public_subnets" {
  type = list
  description = "Seconf subnet for ecternal ALB"
}


variable "ami-nginx" {
  type        = string
  description = "ami for nginx"
}

variable "nginx-alb-tgt" {
  type        = string
  description = "nginx reverse proxy target group"
}

variable "wordpress-alb-tgt" {
  type        = string
  description = "wordpress target group"
}


variable "tooling-alb-tgt" {
  type        = string
  description = "tooling target group"
}


variable "max_size" {
  type        = number
  description = "maximum number for autoscaling"
}

variable "min_size" {
  type        = number
  description = "minimum number for autoscaling"
}

variable "desired_capacity" {
  type        = number
  description = "Desired number of instance in autoscaling group"

}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```

<img width="963" alt="Screenshot 2023-01-18 at 8 44 06 AM" src="https://user-images.githubusercontent.com/105562242/213074713-3ef6a5a1-3849-4015-a61c-a4b8ce1e4875.png">

##### Refactoring the code the root main.tf folder:

```
# creating VPC
module "VPC" {
  source                              = "./modules/VPC"
  region                              = var.region
  vpc_cidr                            = var.vpc_cidr
  enable_dns_support                  = var.enable_dns_support
  enable_dns_hostnames                = var.enable_dns_hostnames
  enable_classiclink                  = var.enable_classiclink
  preferred_number_of_public_subnets  = var.preferred_number_of_public_subnets
  preferred_number_of_private_subnets = var.preferred_number_of_private_subnets
  private_subnets                     = [for i in range(1, 8, 2) : cidrsubnet(var.vpc_cidr, 8, i)]
  public_subnets                      = [for i in range(2, 5, 2) : cidrsubnet(var.vpc_cidr, 8, i)]
}

#Module for Application Load balancer, this will create Extenal Load balancer and internal load balancer
module "ALB" {
  source             = "./modules/ALB"
  name               = "ACS-ext-alb"
  vpc_id             = module.VPC.vpc_id
  public-sg          = module.security.ALB-sg
  private-sg         = module.security.IALB-sg
  public-sbn-1       = module.VPC.public_subnets-1
  public-sbn-2       = module.VPC.public_subnets-2
  private-sbn-1      = module.VPC.private_subnets-1
  private-sbn-2      = module.VPC.private_subnets-2
  load_balancer_type = "application"
  ip_address_type    = "ipv4"
}

module "security" {
  source = "./modules/Security"
  vpc_id = module.VPC.vpc_id
}


module "AutoScaling" {
  source            = "./modules/Autoscaling"
  ami-web           = var.ami
  ami-bastion       = var.ami
  ami-nginx         = var.ami
  desired_capacity  = 2
  min_size          = 2
  max_size          = 2
  web-sg            = [module.security.web-sg]
  bastion-sg        = [module.security.bastion-sg]
  nginx-sg          = [module.security.nginx-sg]
  wordpress-alb-tgt = module.ALB.wordpress-tgt
  nginx-alb-tgt     = module.ALB.nginx-tgt
  tooling-alb-tgt   = module.ALB.tooling-tgt
  instance_profile  = module.VPC.instance_profile
  public_subnets    = [module.VPC.public_subnets-1, module.VPC.public_subnets-2]
  private_subnets   = [module.VPC.private_subnets-1, module.VPC.private_subnets-2]
  keypair           = var.keypair

}

# Module for Elastic Filesystem; this module will creat elastic file system isn the webservers availablity
# zone and allow traffic fro the webservers

module "EFS" {
  source       = "./modules/EFS"
  efs-subnet-1 = module.VPC.private_subnets-1
  efs-subnet-2 = module.VPC.private_subnets-2
  efs-sg       = [module.security.datalayer-sg]
  account_no   = var.account_no
}

# RDS module; this module will create the RDS instance in the private subnet

module "RDS" {
  source          = "./modules/RDS"
  db-password     = var.master-password
  db-username     = var.master-username
  db-sg           = [module.security.datalayer-sg]
  private_subnets = [module.VPC.private_subnets-3, module.VPC.private_subnets-4]
}

# The Module creates instances for jenkins, sonarqube abd jfrog
module "compute" {
  source          = "./modules/compute"
  ami-jenkins     = var.ami
  ami-sonar       = var.ami
  ami-jfrog       = var.ami
  subnets-compute = module.VPC.public_subnets-1
  sg-compute      = [module.security.ALB-sg]
  keypair         = var.keypair
}

```

##### STEP 3: Executing The Terraform Plan
------------------------------------------------------

* To ensure the validation of the whole setup, running the command terraform validate
* Testing the configuration by running the command terraform plan

<img width="1067" alt="Screenshot 2023-01-18 at 8 53 43 AM" src="https://user-images.githubusercontent.com/105562242/213082778-e78c0c45-9152-45e0-84e2-625499f13081.png">

<img width="1072" alt="Screenshot 2023-01-18 at 8 54 05 AM" src="https://user-images.githubusercontent.com/105562242/213082790-878ea13d-9676-48a4-b646-7821cbdf7b40.png">


Acquiring state lock. This may take a few moments...
var.environment
  Enviroment

  Enter a value: 
module.AutoScaling.data.aws_availability_zones.available: Reading...
module.ALB.data.aws_route53_zone.saikat: Reading...
module.VPC.data.aws_availability_zones.available: Reading...
aws_dynamodb_table.terraform_locks: Refreshing state... [id=terraform-locks]
aws_s3_bucket.terraform_state: Refreshing state... [id=project19-bucket-saikat]
module.VPC.data.aws_availability_zones.available: Read complete after 1s [id=us-east-1]
module.AutoScaling.data.aws_availability_zones.available: Read complete after 1s [id=us-east-1]
module.ALB.data.aws_route53_zone.saikat: Read complete after 3s [id=Z031762920PPGKBQDT844]

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.ALB.aws_acm_certificate.saikat will be created
  + resource "aws_acm_certificate" "saikat" {
      + arn                       = (known after apply)
      + domain_name               = "*.saikat-devops.click"
      + domain_validation_options = [
          + {
              + domain_name           = "*.saikat-devops.click"
              + resource_record_name  = (known after apply)
              + resource_record_type  = (known after apply)
              + resource_record_value = (known after apply)
            },
        ]
      + id                        = (known after apply)
      + not_after                 = (known after apply)
      + not_before                = (known after apply)
      + status                    = (known after apply)
      + subject_alternative_names = [
          + "*.saikat-devops.click",
        ]
      + tags_all                  = (known after apply)
      + validation_emails         = (known after apply)
      + validation_method         = "DNS"
    }

  # module.ALB.aws_acm_certificate_validation.saikat will be created
  + resource "aws_acm_certificate_validation" "saikat" {
      + certificate_arn         = (known after apply)
      + id                      = (known after apply)
      + validation_record_fqdns = (known after apply)
    }

  # module.ALB.aws_lb.ext-alb will be created
  + resource "aws_lb" "ext-alb" {
      + arn                        = (known after apply)
      + arn_suffix                 = (known after apply)
      + desync_mitigation_mode     = "defensive"
      + dns_name                   = (known after apply)
      + drop_invalid_header_fields = false
      + enable_deletion_protection = false
      + enable_http2               = true
      + enable_waf_fail_open       = false
      + id                         = (known after apply)
      + idle_timeout               = 60
      + internal                   = false
      + ip_address_type            = "ipv4"
      + load_balancer_type         = "application"
      + name                       = "ACS-ext-alb"
      + preserve_host_header       = false
      + security_groups            = (known after apply)
      + subnets                    = (known after apply)
      + tags                       = {
          + "Name" = "ACS-ext-alb"
        }
      + tags_all                   = {
          + "Name" = "ACS-ext-alb"
        }
      + vpc_id                     = (known after apply)
      + zone_id                    = (known after apply)

      + subnet_mapping {
          + allocation_id        = (known after apply)
          + ipv6_address         = (known after apply)
          + outpost_id           = (known after apply)
          + private_ipv4_address = (known after apply)
          + subnet_id            = (known after apply)
        }
    }

  # module.ALB.aws_lb.ialb will be created
  + resource "aws_lb" "ialb" {
      + arn                        = (known after apply)
      + arn_suffix                 = (known after apply)
      + desync_mitigation_mode     = "defensive"
      + dns_name                   = (known after apply)
      + drop_invalid_header_fields = false
      + enable_deletion_protection = false
      + enable_http2               = true
      + enable_waf_fail_open       = false
      + id                         = (known after apply)
      + idle_timeout               = 60
      + internal                   = true
      + ip_address_type            = "ipv4"
      + load_balancer_type         = "application"
      + name                       = "ialb"
      + preserve_host_header       = false
      + security_groups            = (known after apply)
      + subnets                    = (known after apply)
      + tags                       = {
          + "Name" = "ACS-int-alb"
        }
      + tags_all                   = {
          + "Name" = "ACS-int-alb"
        }
      + vpc_id                     = (known after apply)
      + zone_id                    = (known after apply)

      + subnet_mapping {
          + allocation_id        = (known after apply)
          + ipv6_address         = (known after apply)
          + outpost_id           = (known after apply)
          + private_ipv4_address = (known after apply)
          + subnet_id            = (known after apply)
        }
    }

  # module.ALB.aws_lb_listener.nginx-listner will be created
  + resource "aws_lb_listener" "nginx-listner" {
      + arn               = (known after apply)
      + certificate_arn   = (known after apply)
      + id                = (known after apply)
      + load_balancer_arn = (known after apply)
      + port              = 443
      + protocol          = "HTTPS"
      + ssl_policy        = (known after apply)
      + tags_all          = (known after apply)

      + default_action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }
    }

  # module.ALB.aws_lb_listener.web-listener will be created
  + resource "aws_lb_listener" "web-listener" {
      + arn               = (known after apply)
      + certificate_arn   = (known after apply)
      + id                = (known after apply)
      + load_balancer_arn = (known after apply)
      + port              = 443
      + protocol          = "HTTPS"
      + ssl_policy        = (known after apply)
      + tags_all          = (known after apply)

      + default_action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }
    }

  # module.ALB.aws_lb_listener_rule.tooling-listener will be created
  + resource "aws_lb_listener_rule" "tooling-listener" {
      + arn          = (known after apply)
      + id           = (known after apply)
      + listener_arn = (known after apply)
      + priority     = 99
      + tags_all     = (known after apply)

      + action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }

      + condition {
          + host_header {
              + values = [
                  + "tooling.saikat.co.in",
                ]
            }
        }
    }

  # module.ALB.aws_lb_target_group.nginx-tgt will be created
  + resource "aws_lb_target_group" "nginx-tgt" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = false
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancing_algorithm_type      = (known after apply)
      + name                               = "nginx-tgt"
      + port                               = 443
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTPS"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = (known after apply)

      + health_check {
          + enabled             = true
          + healthy_threshold   = 5
          + interval            = 10
          + matcher             = (known after apply)
          + path                = "/healthstatus"
          + port                = "traffic-port"
          + protocol            = "HTTPS"
          + timeout             = 5
          + unhealthy_threshold = 2
        }

      + stickiness {
          + cookie_duration = (known after apply)
          + cookie_name     = (known after apply)
          + enabled         = (known after apply)
          + type            = (known after apply)
        }
    }

  # module.ALB.aws_lb_target_group.tooling-tgt will be created
  + resource "aws_lb_target_group" "tooling-tgt" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = false
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancing_algorithm_type      = (known after apply)
      + name                               = "saikat-tooling-tgt"
      + port                               = 443
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTPS"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = (known after apply)

      + health_check {
          + enabled             = true
          + healthy_threshold   = 5
          + interval            = 10
          + matcher             = (known after apply)
          + path                = "/healthstatus"
          + port                = "traffic-port"
          + protocol            = "HTTPS"
          + timeout             = 5
          + unhealthy_threshold = 2
        }

      + stickiness {
          + cookie_duration = (known after apply)
          + cookie_name     = (known after apply)
          + enabled         = (known after apply)
          + type            = (known after apply)
        }
    }

  # module.ALB.aws_lb_target_group.wordpress-tgt will be created
  + resource "aws_lb_target_group" "wordpress-tgt" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = false
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancing_algorithm_type      = (known after apply)
      + name                               = "wordpress-tgt"
      + port                               = 443
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTPS"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = (known after apply)

      + health_check {
          + enabled             = true
          + healthy_threshold   = 5
          + interval            = 10
          + matcher             = (known after apply)
          + path                = "/healthstatus"
          + port                = "traffic-port"
          + protocol            = "HTTPS"
          + timeout             = 5
          + unhealthy_threshold = 2
        }

      + stickiness {
          + cookie_duration = (known after apply)
          + cookie_name     = (known after apply)
          + enabled         = (known after apply)
          + type            = (known after apply)
        }
    }

  # module.ALB.aws_route53_record.saikat["*.saikat-devops.click"] will be created
  + resource "aws_route53_record" "saikat" {
      + allow_overwrite = true
      + fqdn            = (known after apply)
      + id              = (known after apply)
      + name            = (known after apply)
      + records         = (known after apply)
      + ttl             = 60
      + type            = (known after apply)
      + zone_id         = "Z031762920PPGKBQDT844"
    }

  # module.ALB.aws_route53_record.tooling will be created
  + resource "aws_route53_record" "tooling" {
      + allow_overwrite = (known after apply)
      + fqdn            = (known after apply)
      + id              = (known after apply)
      + name            = "tooling.saikat-devops.click"
      + type            = "A"
      + zone_id         = "Z031762920PPGKBQDT844"

      + alias {
          + evaluate_target_health = true
          + name                   = (known after apply)
          + zone_id                = (known after apply)
        }
    }

  # module.ALB.aws_route53_record.wordpress will be created
  + resource "aws_route53_record" "wordpress" {
      + allow_overwrite = (known after apply)
      + fqdn            = (known after apply)
      + id              = (known after apply)
      + name            = "wordpress.saikat-devops.click"
      + type            = "A"
      + zone_id         = "Z031762920PPGKBQDT844"

      + alias {
          + evaluate_target_health = true
          + name                   = (known after apply)
          + zone_id                = (known after apply)
        }
    }

  # module.AutoScaling.aws_autoscaling_attachment.asg_attachment_nginx will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
      + autoscaling_group_name = (known after apply)
      + id                     = (known after apply)
      + lb_target_group_arn    = (known after apply)
    }

  # module.AutoScaling.aws_autoscaling_attachment.asg_attachment_tooling will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
      + autoscaling_group_name = (known after apply)
      + id                     = (known after apply)
      + lb_target_group_arn    = (known after apply)
    }

  # module.AutoScaling.aws_autoscaling_attachment.asg_attachment_wordpress will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
      + autoscaling_group_name = (known after apply)
      + id                     = (known after apply)
      + lb_target_group_arn    = (known after apply)
    }

  # module.AutoScaling.aws_autoscaling_group.bastion-asg will be created
  + resource "aws_autoscaling_group" "bastion-asg" {
      + arn                       = (known after apply)
      + availability_zones        = (known after apply)
      + default_cooldown          = (known after apply)
      + desired_capacity          = 2
      + force_delete              = false
      + force_delete_warm_pool    = false
      + health_check_grace_period = 300
      + health_check_type         = "ELB"
      + id                        = (known after apply)
      + max_size                  = 2
      + metrics_granularity       = "1Minute"
      + min_size                  = 2
      + name                      = "bastion-asg"
      + name_prefix               = (known after apply)
      + protect_from_scale_in     = false
      + service_linked_role_arn   = (known after apply)
      + vpc_zone_identifier       = (known after apply)
      + wait_for_capacity_timeout = "10m"

      + launch_template {
          + id      = (known after apply)
          + name    = (known after apply)
          + version = "$Latest"
        }

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "ACS-bastion"
        }
    }

  # module.AutoScaling.aws_autoscaling_group.nginx-asg will be created
  + resource "aws_autoscaling_group" "nginx-asg" {
      + arn                       = (known after apply)
      + availability_zones        = (known after apply)
      + default_cooldown          = (known after apply)
      + desired_capacity          = 1
      + force_delete              = false
      + force_delete_warm_pool    = false
      + health_check_grace_period = 300
      + health_check_type         = "ELB"
      + id                        = (known after apply)
      + max_size                  = 2
      + metrics_granularity       = "1Minute"
      + min_size                  = 1
      + name                      = "nginx-asg"
      + name_prefix               = (known after apply)
      + protect_from_scale_in     = false
      + service_linked_role_arn   = (known after apply)
      + vpc_zone_identifier       = (known after apply)
      + wait_for_capacity_timeout = "10m"

      + launch_template {
          + id      = (known after apply)
          + name    = (known after apply)
          + version = "$Latest"
        }

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "ACS-nginx"
        }
    }

  # module.AutoScaling.aws_autoscaling_group.tooling-asg will be created
  + resource "aws_autoscaling_group" "tooling-asg" {
      + arn                       = (known after apply)
      + availability_zones        = (known after apply)
      + default_cooldown          = (known after apply)
      + desired_capacity          = 2
      + force_delete              = false
      + force_delete_warm_pool    = false
      + health_check_grace_period = 300
      + health_check_type         = "ELB"
      + id                        = (known after apply)
      + max_size                  = 2
      + metrics_granularity       = "1Minute"
      + min_size                  = 2
      + name                      = "tooling-asg"
      + name_prefix               = (known after apply)
      + protect_from_scale_in     = false
      + service_linked_role_arn   = (known after apply)
      + vpc_zone_identifier       = (known after apply)
      + wait_for_capacity_timeout = "10m"

      + launch_template {
          + id      = (known after apply)
          + name    = (known after apply)
          + version = "$Latest"
        }

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "ACS-tooling"
        }
    }

  # module.AutoScaling.aws_autoscaling_group.wordpress-asg will be created
  + resource "aws_autoscaling_group" "wordpress-asg" {
      + arn                       = (known after apply)
      + availability_zones        = (known after apply)
      + default_cooldown          = (known after apply)
      + desired_capacity          = 2
      + force_delete              = false
      + force_delete_warm_pool    = false
      + health_check_grace_period = 300
      + health_check_type         = "ELB"
      + id                        = (known after apply)
      + max_size                  = 2
      + metrics_granularity       = "1Minute"
      + min_size                  = 2
      + name                      = "wordpress-asg"
      + name_prefix               = (known after apply)
      + protect_from_scale_in     = false
      + service_linked_role_arn   = (known after apply)
      + vpc_zone_identifier       = (known after apply)
      + wait_for_capacity_timeout = "10m"

      + launch_template {
          + id      = (known after apply)
          + name    = (known after apply)
          + version = "$Latest"
        }

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "ACS-wordpress"
        }
    }

  # module.AutoScaling.aws_autoscaling_notification.saikat_notifications will be created
  + resource "aws_autoscaling_notification" "saikat_notifications" {
      + group_names   = [
          + "bastion-asg",
          + "nginx-asg",
          + "tooling-asg",
          + "wordpress-asg",
        ]
      + id            = (known after apply)
      + notifications = [
          + "autoscaling:EC2_INSTANCE_LAUNCH",
          + "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
          + "autoscaling:EC2_INSTANCE_TERMINATE",
          + "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
        ]
      + topic_arn     = (known after apply)
    }

  # module.AutoScaling.aws_launch_template.bastion-launch-template will be created
  + resource "aws_launch_template" "bastion-launch-template" {
      + arn                    = (known after apply)
      + default_version        = (known after apply)
      + id                     = (known after apply)
      + image_id               = "ami-09e67e426f25ce0d7"
      + instance_type          = "t2.micro"
      + key_name               = "redhat"
      + latest_version         = (known after apply)
      + name                   = (known after apply)
      + name_prefix            = (known after apply)
      + tags_all               = (known after apply)
      + user_data              = "IyBiYXN0aW9uIHVzZXJkYXRhCiMhL2Jpbi9iYXNoCnl1bSBpbnN0YWxsIC15IG15c3FsCnl1bSBpbnN0YWxsIC15IGdpdCB0bXV4Cnl1bSBpbnN0YWxsIC15IGFuc2libGU="
      + vpc_security_group_ids = (known after apply)

      + iam_instance_profile {
          + name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_protocol_ipv6          = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + placement {
          + availability_zone = "random_shuffle.az_list.result"
        }

      + tag_specifications {
          + resource_type = "instance"
          + tags          = {
              + "Name" = "bastion-launch-template"
            }
        }
    }

  # module.AutoScaling.aws_launch_template.nginx-launch-template will be created
  + resource "aws_launch_template" "nginx-launch-template" {
      + arn                    = (known after apply)
      + default_version        = (known after apply)
      + id                     = (known after apply)
      + image_id               = "ami-09e67e426f25ce0d7"
      + instance_type          = "t2.micro"
      + key_name               = "redhat"
      + latest_version         = (known after apply)
      + name                   = (known after apply)
      + name_prefix            = (known after apply)
      + tags_all               = (known after apply)
      + user_data              = "IyEvYmluL2Jhc2gKeXVtIGluc3RhbGwgLXkgbmdpbngKc3lzdGVtY3RsIHN0YXJ0IG5naW54CnN5c3RlbWN0bCBlbmFibGUgbmdpbngKZ2l0IGNsb25lIGh0dHBzOi8vZ2l0aHViLmNvbS9MaXZpbmdzdG9uZTk1L0FDUy1wcm9qZWN0LWNvbmZpZy5naXQKbXYgL0FDUy1wcm9qZWN0LWNvbmZpZy9yZXZlcnNlLmNvbmYgL2V0Yy9uZ2lueC8KbXYgL2V0Yy9uZ2lueC9uZ2lueC5jb25mIC9ldGMvbmdpbngvbmdpbnguY29uZi1kaXN0cm8KY2QgL2V0Yy9uZ2lueC8KdG91Y2ggbmdpbnguY29uZgpzZWQgLW4gJ3cgbmdpbnguY29uZicgcmV2ZXJzZS5jb25mCnN5c3RlbWN0bCByZXN0YXJ0IG5naW54CnJtIC1yZiByZXZlcnNlLmNvbmYKcm0gLXJmIC9BQ1MtcHJvamVjdC1jb25maWc="
      + vpc_security_group_ids = (known after apply)

      + iam_instance_profile {
          + name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_protocol_ipv6          = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + placement {
          + availability_zone = "random_shuffle.az_list.result"
        }

      + tag_specifications {
          + resource_type = "instance"
          + tags          = {
              + "Name" = "nginx-launch-template"
            }
        }
    }

  # module.AutoScaling.aws_launch_template.tooling-launch-template will be created
  + resource "aws_launch_template" "tooling-launch-template" {
      + arn                    = (known after apply)
      + default_version        = (known after apply)
      + id                     = (known after apply)
      + image_id               = "ami-09e67e426f25ce0d7"
      + instance_type          = "t2.micro"
      + key_name               = "redhat"
      + latest_version         = (known after apply)
      + name                   = (known after apply)
      + name_prefix            = (known after apply)
      + tags_all               = (known after apply)
      + user_data              = "IyEvYmluL2Jhc2gKbWtkaXIgL3Zhci93d3cvCnN1ZG8gbW91bnQgLXQgZWZzIC1vIHRscyxhY2Nlc3Nwb2ludD1mc2FwLTAxYzEzYTQwMTljYTU5ZGJlIGZzLThiNTAxZDNmOi8gL3Zhci93d3cvCnl1bSBpbnN0YWxsIC15IGh0dHBkIApzeXN0ZW1jdGwgc3RhcnQgaHR0cGQKc3lzdGVtY3RsIGVuYWJsZSBodHRwZAp5dW0gbW9kdWxlIHJlc2V0IHBocCAteQp5dW0gbW9kdWxlIGVuYWJsZSBwaHA6cmVtaS03LjQgLXkKeXVtIGluc3RhbGwgLXkgcGhwIHBocC1jb21tb24gcGhwLW1ic3RyaW5nIHBocC1vcGNhY2hlIHBocC1pbnRsIHBocC14bWwgcGhwLWdkIHBocC1jdXJsIHBocC1teXNxbG5kIHBocC1mcG0gcGhwLWpzb24Kc3lzdGVtY3RsIHN0YXJ0IHBocC1mcG0Kc3lzdGVtY3RsIGVuYWJsZSBwaHAtZnBtCmdpdCBjbG9uZSBodHRwczovL2dpdGh1Yi5jb20vTGl2aW5nc3RvbmU5NS90b29saW5nLTEuZ2l0Cm1rZGlyIC92YXIvd3d3L2h0bWwKY3AgLVIgL3Rvb2xpbmctMS9odG1sLyogIC92YXIvd3d3L2h0bWwvCmNkIC90b29saW5nLTEKbXlzcWwgLWggYWNzLWRhdGFiYXNlLmNkcXBiamtldGh2MC51cy1lYXN0LTEucmRzLmFtYXpvbmF3cy5jb20gLXUgQUNTYWRtaW4gLXAgdG9vbGluZ2RiIDwgdG9vbGluZy1kYi5zcWwKY2QgL3Zhci93d3cvaHRtbC8KdG91Y2ggaGVhbHRoc3RhdHVzCnNlZCAtaSAicy8kZGIgPSBteXNxbGlfY29ubmVjdCgnbXlzcWwudG9vbGluZy5zdmMuY2x1c3Rlci5sb2NhbCcsICdhZG1pbicsICdhZG1pbicsICd0b29saW5nJyk7LyRkYiA9IG15c3FsaV9jb25uZWN0KCdhY3MtZGF0YWJhc2UuY2RxcGJqa2V0aHYwLnVzLWVhc3QtMS5yZHMuYW1hem9uYXdzLmNvbSAnLCAnQUNTYWRtaW4nLCAnYWRtaW4xMjM0NScsICd0b29saW5nZGInKTsvZyIgZnVuY3Rpb25zLnBocApjaGNvbiAtdCBodHRwZF9zeXNfcndfY29udGVudF90IC92YXIvd3d3L2h0bWwvIC1SCnN5c3RlbWN0bCByZXN0YXJ0IGh0dHBk"
      + vpc_security_group_ids = (known after apply)

      + iam_instance_profile {
          + name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_protocol_ipv6          = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + placement {
          + availability_zone = "random_shuffle.az_list.result"
        }

      + tag_specifications {
          + resource_type = "instance"
          + tags          = {
              + "Name" = "tooling-launch-template"
            }
        }
    }

  # module.AutoScaling.aws_launch_template.wordpress-launch-template will be created
  + resource "aws_launch_template" "wordpress-launch-template" {
      + arn                    = (known after apply)
      + default_version        = (known after apply)
      + id                     = (known after apply)
      + image_id               = "ami-09e67e426f25ce0d7"
      + instance_type          = "t2.micro"
      + key_name               = "redhat"
      + latest_version         = (known after apply)
      + name                   = (known after apply)
      + name_prefix            = (known after apply)
      + tags_all               = (known after apply)
      + user_data              = "IyEvYmluL2Jhc2gKbWtkaXIgL3Zhci93d3cvCnN1ZG8gbW91bnQgLXQgZWZzIC1vIHRscyxhY2Nlc3Nwb2ludD1mc2FwLTBmOTM2NDY3OTM4M2ZmYmMwIGZzLThiNTAxZDNmOi8gL3Zhci93d3cvCnl1bSBpbnN0YWxsIC15IGh0dHBkIApzeXN0ZW1jdGwgc3RhcnQgaHR0cGQKc3lzdGVtY3RsIGVuYWJsZSBodHRwZAp5dW0gbW9kdWxlIHJlc2V0IHBocCAteQp5dW0gbW9kdWxlIGVuYWJsZSBwaHA6cmVtaS03LjQgLXkKeXVtIGluc3RhbGwgLXkgcGhwIHBocC1jb21tb24gcGhwLW1ic3RyaW5nIHBocC1vcGNhY2hlIHBocC1pbnRsIHBocC14bWwgcGhwLWdkIHBocC1jdXJsIHBocC1teXNxbG5kIHBocC1mcG0gcGhwLWpzb24Kc3lzdGVtY3RsIHN0YXJ0IHBocC1mcG0Kc3lzdGVtY3RsIGVuYWJsZSBwaHAtZnBtCndnZXQgaHR0cDovL3dvcmRwcmVzcy5vcmcvbGF0ZXN0LnRhci5negp0YXIgeHp2ZiBsYXRlc3QudGFyLmd6CnJtIC1yZiBsYXRlc3QudGFyLmd6CmNwIHdvcmRwcmVzcy93cC1jb25maWctc2FtcGxlLnBocCB3b3JkcHJlc3Mvd3AtY29uZmlnLnBocApta2RpciAvdmFyL3d3dy9odG1sLwpjcCAtUiAvd29yZHByZXNzLyogL3Zhci93d3cvaHRtbC8KY2QgL3Zhci93d3cvaHRtbC8KdG91Y2ggaGVhbHRoc3RhdHVzCnNlZCAtaSAicy9sb2NhbGhvc3QvYWNzLWRhdGFiYXNlLmNkcXBiamtldGh2MC51cy1lYXN0LTEucmRzLmFtYXpvbmF3cy5jb20vZyIgd3AtY29uZmlnLnBocCAKc2VkIC1pICJzL3VzZXJuYW1lX2hlcmUvQUNTYWRtaW4vZyIgd3AtY29uZmlnLnBocCAKc2VkIC1pICJzL3Bhc3N3b3JkX2hlcmUvYWRtaW4xMjM0NS9nIiB3cC1jb25maWcucGhwIApzZWQgLWkgInMvZGF0YWJhc2VfbmFtZV9oZXJlL3dvcmRwcmVzc2RiL2ciIHdwLWNvbmZpZy5waHAgCmNoY29uIC10IGh0dHBkX3N5c19yd19jb250ZW50X3QgL3Zhci93d3cvaHRtbC8gLVIKc3lzdGVtY3RsIHJlc3RhcnQgaHR0cGQ="
      + vpc_security_group_ids = (known after apply)

      + iam_instance_profile {
          + name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_protocol_ipv6          = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + placement {
          + availability_zone = "random_shuffle.az_list.result"
        }

      + tag_specifications {
          + resource_type = "instance"
          + tags          = {
              + "Name" = "wordpress-launch-template"
            }
        }
    }

  # module.AutoScaling.aws_sns_topic.ACS-sns will be created
  + resource "aws_sns_topic" "ACS-sns" {
      + arn                         = (known after apply)
      + content_based_deduplication = false
      + fifo_topic                  = false
      + id                          = (known after apply)
      + name                        = "Default_CloudWatch_Alarms_Topic"
      + name_prefix                 = (known after apply)
      + owner                       = (known after apply)
      + policy                      = (known after apply)
      + tags_all                    = (known after apply)
    }

  # module.AutoScaling.random_shuffle.az_list will be created
  + resource "random_shuffle" "az_list" {
      + id     = (known after apply)
      + input  = [
          + "us-east-1a",
          + "us-east-1b",
          + "us-east-1c",
          + "us-east-1d",
          + "us-east-1e",
          + "us-east-1f",
        ]
      + result = (known after apply)
    }

  # module.EFS.aws_efs_access_point.tooling will be created
  + resource "aws_efs_access_point" "tooling" {
      + arn             = (known after apply)
      + file_system_arn = (known after apply)
      + file_system_id  = (known after apply)
      + id              = (known after apply)
      + owner_id        = (known after apply)
      + tags_all        = (known after apply)

      + posix_user {
          + gid = 0
          + uid = 0
        }

      + root_directory {
          + path = "/tooling"

          + creation_info {
              + owner_gid   = 0
              + owner_uid   = 0
              + permissions = "755"
            }
        }
    }

  # module.EFS.aws_efs_access_point.wordpress will be created
  + resource "aws_efs_access_point" "wordpress" {
      + arn             = (known after apply)
      + file_system_arn = (known after apply)
      + file_system_id  = (known after apply)
      + id              = (known after apply)
      + owner_id        = (known after apply)
      + tags_all        = (known after apply)

      + posix_user {
          + gid = 0
          + uid = 0
        }

      + root_directory {
          + path = "/wordpress"

          + creation_info {
              + owner_gid   = 0
              + owner_uid   = 0
              + permissions = "755"
            }
        }
    }

  # module.EFS.aws_efs_file_system.ACS-efs will be created
  + resource "aws_efs_file_system" "ACS-efs" {
      + arn                     = (known after apply)
      + availability_zone_id    = (known after apply)
      + availability_zone_name  = (known after apply)
      + creation_token          = (known after apply)
      + dns_name                = (known after apply)
      + encrypted               = true
      + id                      = (known after apply)
      + kms_key_id              = (known after apply)
      + number_of_mount_targets = (known after apply)
      + owner_id                = (known after apply)
      + performance_mode        = (known after apply)
      + size_in_bytes           = (known after apply)
      + tags                    = {
          + "Name" = "ACS-file-system"
        }
      + tags_all                = {
          + "Name" = "ACS-file-system"
        }
      + throughput_mode         = "bursting"
    }

  # module.EFS.aws_efs_mount_target.subnet-1 will be created
  + resource "aws_efs_mount_target" "subnet-1" {
      + availability_zone_id   = (known after apply)
      + availability_zone_name = (known after apply)
      + dns_name               = (known after apply)
      + file_system_arn        = (known after apply)
      + file_system_id         = (known after apply)
      + id                     = (known after apply)
      + ip_address             = (known after apply)
      + mount_target_dns_name  = (known after apply)
      + network_interface_id   = (known after apply)
      + owner_id               = (known after apply)
      + security_groups        = (known after apply)
      + subnet_id              = (known after apply)
    }

  # module.EFS.aws_efs_mount_target.subnet-2 will be created
  + resource "aws_efs_mount_target" "subnet-2" {
      + availability_zone_id   = (known after apply)
      + availability_zone_name = (known after apply)
      + dns_name               = (known after apply)
      + file_system_arn        = (known after apply)
      + file_system_id         = (known after apply)
      + id                     = (known after apply)
      + ip_address             = (known after apply)
      + mount_target_dns_name  = (known after apply)
      + network_interface_id   = (known after apply)
      + owner_id               = (known after apply)
      + security_groups        = (known after apply)
      + subnet_id              = (known after apply)
    }

  # module.EFS.aws_kms_alias.alias will be created
  + resource "aws_kms_alias" "alias" {
      + arn            = (known after apply)
      + id             = (known after apply)
      + name           = "alias/kms"
      + name_prefix    = (known after apply)
      + target_key_arn = (known after apply)
      + target_key_id  = (known after apply)
    }

  # module.EFS.aws_kms_key.ACS-kms will be created
  + resource "aws_kms_key" "ACS-kms" {
      + arn                                = (known after apply)
      + bypass_policy_lockout_safety_check = false
      + customer_master_key_spec           = "SYMMETRIC_DEFAULT"
      + description                        = "KMS key "
      + enable_key_rotation                = false
      + id                                 = (known after apply)
      + is_enabled                         = true
      + key_id                             = (known after apply)
      + key_usage                          = "ENCRYPT_DECRYPT"
      + multi_region                       = (known after apply)
      + policy                             = jsonencode(
            {
              + Id        = "kms-key-policy"
              + Statement = [
                  + {
                      + Action    = "kms:*"
                      + Effect    = "Allow"
                      + Principal = {
                          + AWS = "arn:aws:iam::811613581700:user/saikat-terraform"
                        }
                      + Resource  = "*"
                      + Sid       = "Enable IAM User Permissions"
                    },
                ]
              + Version   = "2012-10-17"
            }
        )
      + tags_all                           = (known after apply)
    }

  # module.RDS.aws_db_instance.ACS-rds will be created
  + resource "aws_db_instance" "ACS-rds" {
      + address                               = (known after apply)
      + allocated_storage                     = 20
      + apply_immediately                     = (known after apply)
      + arn                                   = (known after apply)
      + auto_minor_version_upgrade            = true
      + availability_zone                     = (known after apply)
      + backup_retention_period               = (known after apply)
      + backup_window                         = (known after apply)
      + ca_cert_identifier                    = (known after apply)
      + character_set_name                    = (known after apply)
      + copy_tags_to_snapshot                 = false
      + db_name                               = "saikatdb"
      + db_subnet_group_name                  = "acs-rds"
      + delete_automated_backups              = true
      + endpoint                              = (known after apply)
      + engine                                = "mysql"
      + engine_version                        = "5.7"
      + engine_version_actual                 = (known after apply)
      + hosted_zone_id                        = (known after apply)
      + id                                    = (known after apply)
      + identifier                            = (known after apply)
      + identifier_prefix                     = (known after apply)
      + instance_class                        = "db.t2.micro"
      + kms_key_id                            = (known after apply)
      + latest_restorable_time                = (known after apply)
      + license_model                         = (known after apply)
      + maintenance_window                    = (known after apply)
      + monitoring_interval                   = 0
      + monitoring_role_arn                   = (known after apply)
      + multi_az                              = true
      + name                                  = (known after apply)
      + nchar_character_set_name              = (known after apply)
      + option_group_name                     = (known after apply)
      + parameter_group_name                  = "default.mysql5.7"
      + password                              = (sensitive value)
      + performance_insights_enabled          = false
      + performance_insights_kms_key_id       = (known after apply)
      + performance_insights_retention_period = (known after apply)
      + port                                  = (known after apply)
      + publicly_accessible                   = false
      + replica_mode                          = (known after apply)
      + replicas                              = (known after apply)
      + resource_id                           = (known after apply)
      + skip_final_snapshot                   = true
      + snapshot_identifier                   = (known after apply)
      + status                                = (known after apply)
      + storage_type                          = "gp2"
      + tags_all                              = (known after apply)
      + timezone                              = (known after apply)
      + username                              = "saikat"
      + vpc_security_group_ids                = (known after apply)
    }

  # module.RDS.aws_db_subnet_group.ACS-rds will be created
  + resource "aws_db_subnet_group" "ACS-rds" {
      + arn         = (known after apply)
      + description = "Managed by Terraform"
      + id          = (known after apply)
      + name        = "acs-rds"
      + name_prefix = (known after apply)
      + subnet_ids  = (known after apply)
      + tags        = {
          + "Name" = "ACS-database"
        }
      + tags_all    = {
          + "Name" = "ACS-database"
        }
    }

  # module.VPC.aws_eip.nat_eip will be created
  + resource "aws_eip" "nat_eip" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags                 = {
          + "Name" = "ACS-EIP-true"
        }
      + tags_all             = {
          + "Name" = "ACS-EIP-true"
        }
      + vpc                  = true
    }

  # module.VPC.aws_iam_instance_profile.ip will be created
  + resource "aws_iam_instance_profile" "ip" {
      + arn         = (known after apply)
      + create_date = (known after apply)
      + id          = (known after apply)
      + name        = "aws_instance_profile_test"
      + path        = "/"
      + role        = "ec2_instance_role"
      + tags_all    = (known after apply)
      + unique_id   = (known after apply)
    }

  # module.VPC.aws_iam_policy.policy will be created
  + resource "aws_iam_policy" "policy" {
      + arn         = (known after apply)
      + description = "A test policy"
      + id          = (known after apply)
      + name        = "ec2_instance_policy"
      + path        = "/"
      + policy      = jsonencode(
            {
              + Statement = [
                  + {
                      + Action   = [
                          + "ec2:Describe*",
                        ]
                      + Effect   = "Allow"
                      + Resource = "*"
                    },
                ]
              + Version   = "2012-10-17"
            }
        )
      + policy_id   = (known after apply)
      + tags        = {
          + "Environment" = "true"
          + "Name"        = "aws assume policy"
        }
      + tags_all    = {
          + "Environment" = "true"
          + "Name"        = "aws assume policy"
        }
    }

  # module.VPC.aws_iam_role.ec2_instance_role will be created
  + resource "aws_iam_role" "ec2_instance_role" {
      + arn                   = (known after apply)
      + assume_role_policy    = jsonencode(
            {
              + Statement = [
                  + {
                      + Action    = "sts:AssumeRole"
                      + Effect    = "Allow"
                      + Principal = {
                          + Service = "ec2.amazonaws.com"
                        }
                      + Sid       = ""
                    },
                ]
              + Version   = "2012-10-17"
            }
        )
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = "ec2_instance_role"
      + name_prefix           = (known after apply)
      + path                  = "/"
      + tags                  = {
          + "Environment" = "true"
          + "Name"        = "aws assume role"
        }
      + tags_all              = {
          + "Environment" = "true"
          + "Name"        = "aws assume role"
        }
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # module.VPC.aws_iam_role_policy_attachment.test-attach will be created
  + resource "aws_iam_role_policy_attachment" "test-attach" {
      + id         = (known after apply)
      + policy_arn = (known after apply)
      + role       = "ec2_instance_role"
    }

  # module.VPC.aws_internet_gateway.ig will be created
  + resource "aws_internet_gateway" "ig" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = (known after apply)
      + tags_all = (known after apply)
      + vpc_id   = (known after apply)
    }

  # module.VPC.aws_nat_gateway.nat will be created
  + resource "aws_nat_gateway" "nat" {
      + allocation_id        = (known after apply)
      + connectivity_type    = "public"
      + id                   = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip           = (known after apply)
      + public_ip            = (known after apply)
      + subnet_id            = (known after apply)
      + tags                 = {
          + "Name" = "ACS-Nat-true"
        }
      + tags_all             = {
          + "Name" = "ACS-Nat-true"
        }
    }

  # module.VPC.aws_route.private-rtb-route will be created
  + resource "aws_route" "private-rtb-route" {
      + destination_cidr_block = "0.0.0.0/0"
      + gateway_id             = (known after apply)
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)
    }

  # module.VPC.aws_route.public-rtb-route will be created
  + resource "aws_route" "public-rtb-route" {
      + destination_cidr_block = "0.0.0.0/0"
      + gateway_id             = (known after apply)
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)
    }

  # module.VPC.aws_route_table.private-rtb will be created
  + resource "aws_route_table" "private-rtb" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Name" = "ACS-Private-Route-Table-true"
        }
      + tags_all         = {
          + "Name" = "ACS-Private-Route-Table-true"
        }
      + vpc_id           = (known after apply)
    }

  # module.VPC.aws_route_table.public-rtb will be created
  + resource "aws_route_table" "public-rtb" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Name" = "ACS-Public-Route-Table-true"
        }
      + tags_all         = {
          + "Name" = "ACS-Public-Route-Table-true"
        }
      + vpc_id           = (known after apply)
    }

  # module.VPC.aws_route_table_association.private-subnets-assoc[0] will be created
  + resource "aws_route_table_association" "private-subnets-assoc" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.VPC.aws_route_table_association.private-subnets-assoc[1] will be created
  + resource "aws_route_table_association" "private-subnets-assoc" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.VPC.aws_route_table_association.private-subnets-assoc[2] will be created
  + resource "aws_route_table_association" "private-subnets-assoc" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.VPC.aws_route_table_association.private-subnets-assoc[3] will be created
  + resource "aws_route_table_association" "private-subnets-assoc" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.VPC.aws_route_table_association.public-subnets-assoc[0] will be created
  + resource "aws_route_table_association" "public-subnets-assoc" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.VPC.aws_route_table_association.public-subnets-assoc[1] will be created
  + resource "aws_route_table_association" "public-subnets-assoc" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.VPC.aws_subnet.private[0] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.1.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "ACS-PrivateSubnet-0"
        }
      + tags_all                                       = {
          + "Name" = "ACS-PrivateSubnet-0"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.VPC.aws_subnet.private[1] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.3.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "ACS-PrivateSubnet-1"
        }
      + tags_all                                       = {
          + "Name" = "ACS-PrivateSubnet-1"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.VPC.aws_subnet.private[2] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1c"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.5.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "ACS-PrivateSubnet-2"
        }
      + tags_all                                       = {
          + "Name" = "ACS-PrivateSubnet-2"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.VPC.aws_subnet.private[3] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1d"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.7.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "ACS-PrivateSubnet-3"
        }
      + tags_all                                       = {
          + "Name" = "ACS-PrivateSubnet-3"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.VPC.aws_subnet.public[0] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.2.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "ACS-PublicSubnet-0"
        }
      + tags_all                                       = {
          + "Name" = "ACS-PublicSubnet-0"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.VPC.aws_subnet.public[1] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.4.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "ACS-PublicSubnet-1"
        }
      + tags_all                                       = {
          + "Name" = "ACS-PublicSubnet-1"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.VPC.aws_vpc.main will be created
  + resource "aws_vpc" "main" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = false
      + enable_classiclink_dns_support       = true
      + enable_dns_hostnames                 = true
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name" = "ACS-VPC"
        }
      + tags_all                             = {
          + "Name" = "ACS-VPC"
        }
    }

  # module.compute.aws_instance.Jenkins will be created
  + resource "aws_instance" "Jenkins" {
      + ami                                  = "ami-09e67e426f25ce0d7"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = true
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "redhat"
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "ACS-Jenkins"
        }
      + tags_all                             = {
          + "Name" = "ACS-Jenkins"
        }
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id                 = (known after apply)
              + capacity_reservation_resource_group_arn = (known after apply)
            }
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + maintenance_options {
          + auto_recovery = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_card_index    = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + private_dns_name_options {
          + enable_resource_name_dns_a_record    = (known after apply)
          + enable_resource_name_dns_aaaa_record = (known after apply)
          + hostname_type                        = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # module.compute.aws_instance.artifactory will be created
  + resource "aws_instance" "artifactory" {
      + ami                                  = "ami-09e67e426f25ce0d7"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = true
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.medium"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "redhat"
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "ACS-artifactory"
        }
      + tags_all                             = {
          + "Name" = "ACS-artifactory"
        }
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id                 = (known after apply)
              + capacity_reservation_resource_group_arn = (known after apply)
            }
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + maintenance_options {
          + auto_recovery = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_card_index    = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + private_dns_name_options {
          + enable_resource_name_dns_a_record    = (known after apply)
          + enable_resource_name_dns_aaaa_record = (known after apply)
          + hostname_type                        = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # module.compute.aws_instance.sonbarqube will be created
  + resource "aws_instance" "sonbarqube" {
      + ami                                  = "ami-09e67e426f25ce0d7"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = true
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.medium"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "redhat"
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "ACS-sonbarqube"
        }
      + tags_all                             = {
          + "Name" = "ACS-sonbarqube"
        }
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id                 = (known after apply)
              + capacity_reservation_resource_group_arn = (known after apply)
            }
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + maintenance_options {
          + auto_recovery = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_card_index    = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + private_dns_name_options {
          + enable_resource_name_dns_a_record    = (known after apply)
          + enable_resource_name_dns_aaaa_record = (known after apply)
          + hostname_type                        = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

  # module.security.aws_security_group.ACS["bastion-sg"] will be created
  + resource "aws_security_group" "ACS" {
      + arn                    = (known after apply)
      + description            = "for bastion instances"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "bastion-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "bastion-sg"
        }
      + tags_all               = {
          + "Name" = "bastion-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # module.security.aws_security_group.ACS["datalayer-sg"] will be created
  + resource "aws_security_group" "ACS" {
      + arn                    = (known after apply)
      + description            = "data layer security group"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "datalayer-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "datalayer-sg"
        }
      + tags_all               = {
          + "Name" = "datalayer-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # module.security.aws_security_group.ACS["ext-alb-sg"] will be created
  + resource "aws_security_group" "ACS" {
      + arn                    = (known after apply)
      + description            = "for external loadbalncer"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "ext-alb-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "ext-alb-sg"
        }
      + tags_all               = {
          + "Name" = "ext-alb-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # module.security.aws_security_group.ACS["int-alb-sg"] will be created
  + resource "aws_security_group" "ACS" {
      + arn                    = (known after apply)
      + description            = "IALB security group"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "int-alb-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "int-alb-sg"
        }
      + tags_all               = {
          + "Name" = "int-alb-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # module.security.aws_security_group.ACS["nginx-sg"] will be created
  + resource "aws_security_group" "ACS" {
      + arn                    = (known after apply)
      + description            = "nginx instances"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "nginx-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "nginx-sg"
        }
      + tags_all               = {
          + "Name" = "nginx-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # module.security.aws_security_group.ACS["webserver-sg"] will be created
  + resource "aws_security_group" "ACS" {
      + arn                    = (known after apply)
      + description            = "webservers security group"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "webserver-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Name" = "webserver-sg"
        }
      + tags_all               = {
          + "Name" = "webserver-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # module.security.aws_security_group_rule.inbound-alb-http will be created
  + resource "aws_security_group_rule" "inbound-alb-http" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 80
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 80
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-alb-https will be created
  + resource "aws_security_group_rule" "inbound-alb-https" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 443
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 443
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-bastion-ssh will be created
  + resource "aws_security_group_rule" "inbound-bastion-ssh" {
      + from_port                = 22
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 22
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-ialb-https will be created
  + resource "aws_security_group_rule" "inbound-ialb-https" {
      + from_port                = 443
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 443
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-mysql-bastion will be created
  + resource "aws_security_group_rule" "inbound-mysql-bastion" {
      + from_port                = 3306
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 3306
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-mysql-webserver will be created
  + resource "aws_security_group_rule" "inbound-mysql-webserver" {
      + from_port                = 3306
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 3306
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-nfs-port will be created
  + resource "aws_security_group_rule" "inbound-nfs-port" {
      + from_port                = 2049
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 2049
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-nginx-http will be created
  + resource "aws_security_group_rule" "inbound-nginx-http" {
      + from_port                = 443
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 443
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-ssh-bastion will be created
  + resource "aws_security_group_rule" "inbound-ssh-bastion" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 22
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 22
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-web-https will be created
  + resource "aws_security_group_rule" "inbound-web-https" {
      + from_port                = 443
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 443
      + type                     = "ingress"
    }

  # module.security.aws_security_group_rule.inbound-web-ssh will be created
  + resource "aws_security_group_rule" "inbound-web-ssh" {
      + from_port                = 22
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 22
      + type                     = "ingress"
    }

Plan: 80 to add, 0 to change, 0 to destroy.

Warning: Argument is deprecated

  with aws_s3_bucket.terraform_state,
  on backend.tf line 1, in resource "aws_s3_bucket" "terraform_state":
   1: resource "aws_s3_bucket" "terraform_state" {

Use the aws_s3_bucket_versioning resource instead

(and 3 more similar warnings elsewhere)

─────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
Releasing state lock. This may take a few moments...

