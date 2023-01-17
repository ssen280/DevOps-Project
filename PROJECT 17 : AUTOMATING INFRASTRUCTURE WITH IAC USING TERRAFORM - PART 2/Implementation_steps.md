
#### INTRODUCTION
--------------------------------------

In continuation to project 16, the remaining resources are created in this project in order to set up a secured infrastructure with Terraform
Code repo : https://github.com/ssen280/PROJECT17-TERRAFORM-CODE.git
--------------------------------------

##### STEP 1: Creating Private Subnet
--------------------------------------
* Due to the AZ of eu-central-1 region is not up to 4 which will return error since it is 4 private subnet that is needed, therefore random_shuffle resource is introduced and then specifying the maximum subnet:

```
resource "random_shuffle" "az_list" {
  input        = data.aws_availability_zones.available.names
  result_count = var.max_subnets
}

```
* Creating the private subnet

```
# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
  map_public_ip_on_launch = true
  availability_zone       = random_shuffle.az_list.result[count.index]

  tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index)
    }
  )
}

```
##### STEP 2: Creating Internet Gateway
--------------------------------------------
* Creating a file called internet_gateway.tf and entering the following codes:

```
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.main.id,"IG")
    } 
  )
}
```

##### STEP 3: Creating NAT Gateway
-------------------------------------------
* Creating a file called nat_gateway.tf and entering the following codes to create a NAT gateway and assign an elastic IP to it:

```
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}

```
##### STEP 4: Creating Routes
----------------------------------------
* Creating a file called route_tables.tf and entering the following codes to create routes for both public and private subnets:

```
# create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table", var.name)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "private-rtb-route" {
  route_table_id         = aws_route_table.private-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_nat_gateway.nat.id
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}

```
##### STEP 5: Creating IAM Roles
---------------------------------------
* An IAM role is passed to the EC2 instances to give them access to some specific resources by first of all creating an AssumeRole and AssumeRole policy.
* Creating a file named roles.tf and entering the following codes:

```
resource "aws_iam_role" "ec2_instance_role" {
name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}

```
* Creating an IAM policy which allows an IAM role to perform action describe to EC2 instances:

```
resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name =  "aws assume policy"
    },
  )

}
```
* Attaching the policy to the IAM role created:

```
resource "aws_iam_role_policy_attachment" "test-attach" {
        role       = aws_iam_role.ec2_instance_role.name
        policy_arn = aws_iam_policy.policy.arn
    }
```
* Creating an Instance Profile and interpolating the IAM Role

```
 resource "aws_iam_instance_profile" "ip" {
        name = "aws_instance_profile_test"
        role =  aws_iam_role.ec2_instance_role.name
    }
```
##### STEP 6: Creating Security Groups
---------------------------------------------
* Creating a new file and called security.tf and entering the following commands to create security group for the Internal and External load balancer, the bastion server, Nginx server, the tooling and wordpress webserver and the data layer:

```
# security group for alb, to allow acess from any where for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

 tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    },
  )

}

# security group for bastion, to allow access into the bastion host from you IP
resource "aws_security_group" "bastion_sg" {
  name        = "vpc_web_sg"
  vpc_id = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

   tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}

#security group for nginx reverse proxy, to allow access only from the extaernal load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

   tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "my-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "my-asg-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

 tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

```

* The aws_security_group_rule is used to reference another security group in a security group.

##### STEP 7: Creating Certificate From Amazon Certificate Manager
------------------------------------------------------------------------
* Creating a new file called cert.tf and entering the following codes to create and validate a certificate AWS:

```
# The entire section create a certiface, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in somexdev.ga
resource "aws_acm_certificate" "somdev" {
  domain_name       = "*.somdev.ga"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "somdev" {
  name         = "somexdev.ga"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "somdev" {
  for_each = {
    for dvo in aws_acm_certificate.somdev.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = false
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.somdev.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "somdev" {
  certificate_arn         = aws_acm_certificate.somdev.arn
  validation_record_fqdns = [for record in aws_route53_record.somdev : record.fqdn]

   timeouts {
    create = "60m"
  }
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.somdev.zone_id
  name    = "tooling.somdev.ga"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.somdev.zone_id
  name    = "wordpress.somdev.ga"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

```
##### STEP 8: Creating Application Load Balancer
-----------------------------------------------------------
* Creating a file called alb.tf
* Entering the following codes to create external(internet-facing) load balancer which balances traffic for the Nginx servers:

```
resource "aws_lb" "ext-alb" {
  name     = "ext-alb"
  internal = false
  security_groups = [
    aws_security_group.ext-alb-sg.id,
  ]

  subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "ACS-ext-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```
* Creating a Target Group for the Nginx server which informs the ALB where to route the traffic:

```
# create Target group for nginx
resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```
* Creating a listener for the Nginx target group:

```
# create a listener for nginx
resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.somdev.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}
```
* Creating a file called output.tf and entering the following codes which outputs the DNS name of the external load balancer

```
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}

```
* In the alb.tf file, entering the following codes to create an internal Application Load Balancer:

```
#Internal Load Balancers for webservers
resource "aws_lb" "ialb" {
  name     = "ialb"
  internal = true
  security_groups = [
    aws_security_group.int-alb-sg.id,
  ]

  subnets = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "ACS-int-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}

```
* Creating the Target Group for the Wordpress and Tooling server to inform the ALB where to route the traffic:

```
# --- target group  for wordpress -------

resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group for tooling -------

resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

```
* Creating a listener for these Target Group:

```
# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes

resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.somdev.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target

resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.somdev.ga"]
    }
  }
}

```
##### STEP 9: Creating An Auto Scaling Group
-------------------------------------------------------
* Creating a file called asg-bastion-nginx.tf which will be used to create an auto scaling group for the bastion and the Nginx server.
* Entering the following codes which creates notification for all the auto scaling group:

```
#### creating sns topic for all the auto scaling groups
resource "aws_sns_topic" "somex-sns" {
  name = "Default_CloudWatch_Alarms_Topic"
}

resource "aws_autoscaling_notification" "somex_notifications" {
  group_names = [
    aws_autoscaling_group.bastion-asg.name,
    aws_autoscaling_group.nginx-asg.name,
    aws_autoscaling_group.wordpress-asg.name,
    aws_autoscaling_group.tooling-asg.name,
  ]
  notifications = [
    "autoscaling:EC2_INSTANCE_LAUNCH",
    "autoscaling:EC2_INSTANCE_TERMINATE",
    "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
    "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
  ]

  topic_arn = aws_sns_topic.somex-sns.arn
}

# Create Launch Template for bastion
resource "aws_launch_template" "bastion-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.bastion_sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "bastion-launch-template"
      },
    )
  }

  user_data = filebase64("${path.module}/bastion.sh")
}

# ---- Autoscaling for bastion  hosts

resource "aws_autoscaling_group" "bastion-asg" {
  name                      = "bastion-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.bastion-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "bastion-launch-template"
    propagate_at_launch = true
  }

}

# launch template for nginx

resource "aws_launch_template" "nginx-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.nginx-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "nginx-launch-template"
      },
    )
  }

  user_data = filebase64("${path.module}/nginx.sh")
}

# ------ Autoscslaling group for reverse proxy nginx ---------

resource "aws_autoscaling_group" "nginx-asg" {
  name                      = "nginx-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.nginx-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "nginx-launch-template"
    propagate_at_launch = true
  }

}

# attaching autoscaling group of nginx to external load balancer
resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
  autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
  lb_target_group_arn    = aws_lb_target_group.nginx-tgt.arn
}

```
* Creating a new file called asg-wordpress-tooling.tf and entering the following codes which creates launch templates and Auto Scaling Group for both the wordpress and tooling webserver:

```
# launch template for wordpress

resource "aws_launch_template" "wordpress-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "wordpress-launch-template"
      },
    )

  }

  user_data = filebase64("${path.module}/wordpress.sh")
}

# ---- Autoscaling for wordpress application

resource "aws_autoscaling_group" "wordpress-asg" {
  name                      = "wordpress-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1
  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.wordpress-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "wordpress-asg"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  wordpress application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
  autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
  lb_target_group_arn    = aws_lb_target_group.wordpress-tgt.arn
}

# launch template for toooling
resource "aws_launch_template" "tooling-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "tooling-launch-template"
      },
    )

  }
  user_data = filebase64("${path.module}/tooling.sh")
}

# ---- Autoscaling for tooling -----

resource "aws_autoscaling_group" "tooling-asg" {
  name                      = "tooling-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.tooling-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "tooling-launch-template"
    propagate_at_launch = true
  }
}
# attaching autoscaling group of  tooling application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
  autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
  lb_target_group_arn    = aws_lb_target_group.tooling-tgt.arn
}

```
* Creating four files that will be used as user data for launching the four different servers:

###### For the bastion server bastion.sh
```
#!/bin/bash 
yum install -y mysql 
yum install -y 
git tmux 
yum install -y ansible
```
###### For the Nginx server nginx.sh
```
#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/somex6/ACS-project-config.git
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config

```
###### For the Wordpress server wordpress.sh

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0f9364679383ffbc0 fs-8b501d3f:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/ACSadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```
###### For Tooling Webserver
```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-01c13a4019ca59dbe fs-8b501d3f:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/Livingstone95/tooling-1.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u ACSadmin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com', 'ACSadmin', 'admin12345', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```
#### STEP 10: Creating Database And EFS Rwsources
-------------------------------------------------------

* Creating a new file called efs.tf.
* Entering the following code to create KMS key to encrypt EFS resource:

```
# create key from key management system
resource "aws_kms_key" "ACS-kms" {
  description = "KMS key "
  policy      = <<EOF
  {
  "Version": "2012-10-17",
  "Id": "kms-key-policy",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::${var.account_no}:user/somex-terraform" },
      "Action": "kms:*",
      "Resource": "*"
    }
  ]
}
EOF
}
```
* Adding the following code which creates EFS, creates access point and mounts it to right subnet:

```
# create key alias
resource "aws_kms_alias" "alias" {
  name          = "alias/kms"
  target_key_id = aws_kms_key.ACS-kms.key_id
}

# create Elastic file system
resource "aws_efs_file_system" "ACS-efs" {
  encrypted  = true
  kms_key_id = aws_kms_key.ACS-kms.arn

  tags = merge(
    var.tags,
    {
      Name = "ACS-efs"
    },
  )
}

# set first mount target for the EFS 
resource "aws_efs_mount_target" "subnet-1" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[2].id
  security_groups = [aws_security_group.datalayer-sg.id]
}

# set second mount target for the EFS 
resource "aws_efs_mount_target" "subnet-2" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[3].id
  security_groups = [aws_security_group.datalayer-sg.id]
}

# create access point for wordpress
resource "aws_efs_access_point" "wordpress" {
  file_system_id = aws_efs_file_system.ACS-efs.id

  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {
    path = "/wordpress"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }

}

# create access point for tooling
resource "aws_efs_access_point" "tooling" {
  file_system_id = aws_efs_file_system.ACS-efs.id
  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {

    path = "/tooling"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }
}
```
* Creating a new file called rds.tf and entering the following code which creates MySQL Relational Database System:

```
# This section will create the subnet group for the RDS  instance using the private subnet
resource "aws_db_subnet_group" "ACS-rds" {
  name       = "acs-rds"
  subnet_ids = [aws_subnet.private[2].id, aws_subnet.private[3].id]

  tags = merge(
    var.tags,
    {
      Name = "ACS-rds"
    },
  )
}

# create the RDS instance with the subnets group
resource "aws_db_instance" "ACS-rds" {
  allocated_storage      = 20
  storage_type           = "gp2"
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = "db.t2.micro"
  db_name                = "somex"
  username               = var.master-username
  password               = var.master-password
  parameter_group_name   = "default.mysql5.7"
  db_subnet_group_name   = aws_db_subnet_group.ACS-rds.name
  skip_final_snapshot    = true
  vpc_security_group_ids = [aws_security_group.datalayer-sg.id]
  multi_az               = "true"
}
```

#### STEP 12: Executing Terraform Apply
------------------------------------------------------
```
saikatsen@Saikats-MacBook-Pro PBL17 % terraform plan
data.aws_availability_zones.available: Reading...
data.aws_route53_zone.saikat: Reading...
aws_iam_policy.policy: Refreshing state... [id=arn:aws:iam::811613581700:policy/ec2_instance_policy]
aws_kms_key.ACS-kms: Refreshing state... [id=b9a0fce6-c0fc-4a97-9797-c534509cea04]
aws_vpc.main: Refreshing state... [id=vpc-0c4013f17b37a5213]
aws_sns_topic.saikat-sns: Refreshing state... [id=arn:aws:sns:us-east-1:811613581700:Default_CloudWatch_Alarms_Topic]
aws_acm_certificate.saikat: Refreshing state... [id=arn:aws:acm:us-east-1:811613581700:certificate/4a30b558-914c-4440-98a6-ab08c7bc4896]
aws_iam_role.ec2_instance_role: Refreshing state... [id=ec2_instance_role]
data.aws_availability_zones.available: Read complete after 0s [id=us-east-1]
random_shuffle.az_list: Refreshing state... [id=-]
data.aws_route53_zone.saikat: Read complete after 2s [id=Z031762920PPGKBQDT844]
aws_route53_record.saikat["*.saikat-devops.click"]: Refreshing state... [id=Z031762920PPGKBQDT844__0bd2331c8182b3f520abc0aa54564b49.saikat-devops.click._CNAME]
aws_iam_role_policy_attachment.test-attach: Refreshing state... [id=ec2_instance_role-20230117014752970100000002]
aws_iam_instance_profile.ip: Refreshing state... [id=aws_instance_profile_test]
aws_kms_alias.alias: Refreshing state... [id=alias/kms]
aws_efs_file_system.ACS-efs: Refreshing state... [id=fs-0cccbe2987395f177]
aws_acm_certificate_validation.saikat: Refreshing state... [id=2023-01-17 01:48:15.375 +0000 UTC]
aws_internet_gateway.ig: Refreshing state... [id=igw-0bebf379fc11e7089]
aws_subnet.private[3]: Refreshing state... [id=subnet-0bb4717832bff519e]
aws_security_group.bastion_sg: Refreshing state... [id=sg-030075847036d548f]
aws_subnet.private[2]: Refreshing state... [id=subnet-09caf8712b32dc16e]
aws_subnet.private[0]: Refreshing state... [id=subnet-0bb4cef51b048e914]
aws_route_table.private-rtb: Refreshing state... [id=rtb-05f0f8c1077eb3db5]
aws_subnet.private[1]: Refreshing state... [id=subnet-0e4f4ef045734903f]
aws_security_group.int-alb-sg: Refreshing state... [id=sg-0a45b7b95e295041d]
aws_lb_target_group.tooling-tgt: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/saikat-tooling-tgt/dfe733266106ad8c]
aws_security_group.webserver-sg: Refreshing state... [id=sg-08a7f6722841d0660]
aws_security_group.datalayer-sg: Refreshing state... [id=sg-04f05f281781252c1]
aws_subnet.public[0]: Refreshing state... [id=subnet-0a8c1e9ff8151ed42]
aws_subnet.public[1]: Refreshing state... [id=subnet-039b237d2b39f0ee7]
aws_lb_target_group.nginx-tgt: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/nginx-tgt/68c93702c4ee59ef]
aws_route_table.public-rtb: Refreshing state... [id=rtb-040778f31bf74ac86]
aws_lb_target_group.wordpress-tgt: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/wordpress-tgt/33f4fcd5bfc27e52]
aws_security_group.nginx-sg: Refreshing state... [id=sg-042d25e10d2e9204d]
aws_security_group.ext-alb-sg: Refreshing state... [id=sg-09c20452387bc6a46]
aws_eip.nat_eip: Refreshing state... [id=eipalloc-052e1b8366c110276]
aws_db_subnet_group.ACS-rds: Refreshing state... [id=acs-rds]
aws_route_table_association.private-subnets-assoc[3]: Refreshing state... [id=rtbassoc-031924301c8778bda]
aws_route_table_association.private-subnets-assoc[2]: Refreshing state... [id=rtbassoc-09d680545f0c0de0c]
aws_route_table_association.private-subnets-assoc[1]: Refreshing state... [id=rtbassoc-0eb33900620535f83]
aws_route_table_association.private-subnets-assoc[0]: Refreshing state... [id=rtbassoc-06c43cd86938cf1ed]
aws_launch_template.bastion-launch-template: Refreshing state... [id=lt-0ae167430fcc0dad7]
aws_lb.ialb: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:loadbalancer/app/ialb/3b6916006dd2283c]
aws_launch_template.wordpress-launch-template: Refreshing state... [id=lt-01c570df5ee40218d]
aws_security_group_rule.inbound-web-https: Refreshing state... [id=sgrule-1091876913]
aws_security_group_rule.inbound-web-ssh: Refreshing state... [id=sgrule-2958450141]
aws_launch_template.tooling-launch-template: Refreshing state... [id=lt-0f4ea7509e95088b3]
aws_efs_access_point.wordpress: Refreshing state... [id=fsap-05c2f42e7607c5b18]
aws_efs_access_point.tooling: Refreshing state... [id=fsap-0c57dbaadaa473183]
aws_efs_mount_target.subnet-1: Refreshing state... [id=fsmt-09c9076f2e76b778c]
aws_security_group_rule.inbound-mysql-bastion: Refreshing state... [id=sgrule-2655927309]
aws_security_group_rule.inbound-mysql-webserver: Refreshing state... [id=sgrule-2119730572]
aws_security_group_rule.inbound-nfs-port: Refreshing state... [id=sgrule-3986560262]
aws_efs_mount_target.subnet-2: Refreshing state... [id=fsmt-0f3ca87dfa14bdd08]
aws_route.public-rtb-route: Refreshing state... [id=r-rtb-040778f31bf74ac861080289494]
aws_route_table_association.public-subnets-assoc[0]: Refreshing state... [id=rtbassoc-04164cc105cffd9bc]
aws_route_table_association.public-subnets-assoc[1]: Refreshing state... [id=rtbassoc-0f7caee9f51c6bc6f]
aws_security_group_rule.inbound-bastion-ssh: Refreshing state... [id=sgrule-3488766914]
aws_launch_template.nginx-launch-template: Refreshing state... [id=lt-02070bc11a825c366]
aws_security_group_rule.inbound-ialb-https: Refreshing state... [id=sgrule-3710512551]
aws_security_group_rule.inbound-nginx-http: Refreshing state... [id=sgrule-4143344385]
aws_lb.ext-alb: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:loadbalancer/app/ext-alb/e1e8d6e3e203fe45]
aws_nat_gateway.nat: Refreshing state... [id=nat-0fcba80facdd222bd]
aws_autoscaling_group.bastion-asg: Refreshing state... [id=bastion-asg]
aws_autoscaling_group.tooling-asg: Refreshing state... [id=tooling-asg]
aws_autoscaling_group.wordpress-asg: Refreshing state... [id=wordpress-asg]
aws_db_instance.ACS-rds: Refreshing state... [id=terraform-2023011701482502350000000b]
aws_lb_listener.web-listener: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:listener/app/ialb/3b6916006dd2283c/07f12dd7a030af94]
aws_autoscaling_group.nginx-asg: Refreshing state... [id=nginx-asg]
aws_lb_listener_rule.tooling-listener: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:listener-rule/app/ialb/3b6916006dd2283c/07f12dd7a030af94/29e022b6d8681367]
aws_route53_record.tooling: Refreshing state... [id=Z031762920PPGKBQDT844_tooling.saikat-devops.click_A]
aws_route53_record.wordpress: Refreshing state... [id=Z031762920PPGKBQDT844_wordpress.saikat-devops.click_A]
aws_lb_listener.nginx-listner: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:listener/app/ext-alb/e1e8d6e3e203fe45/1d4ee547b3a7df36]
aws_route.private-rtb-route: Refreshing state... [id=r-rtb-05f0f8c1077eb3db51080289494]
aws_autoscaling_attachment.asg_attachment_tooling: Refreshing state... [id=tooling-asg-20230117021237609700000005]
aws_autoscaling_attachment.asg_attachment_wordpress: Refreshing state... [id=wordpress-asg-20230117021237629100000006]
aws_autoscaling_notification.saikat_notifications: Refreshing state... [id=arn:aws:sns:us-east-1:811613581700:Default_CloudWatch_Alarms_Topic]
aws_autoscaling_attachment.asg_attachment_nginx: Refreshing state... [id=nginx-asg-20230117021310982700000007]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create
  ~ update in-place

Terraform will perform the following actions:

  # aws_autoscaling_attachment.asg_attachment_nginx will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
      + autoscaling_group_name = "nginx-asg"
      + id                     = (known after apply)
      + lb_target_group_arn    = "arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/nginx-tgt/68c93702c4ee59ef"
    }

  # aws_autoscaling_attachment.asg_attachment_tooling will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
      + autoscaling_group_name = "tooling-asg"
      + id                     = (known after apply)
      + lb_target_group_arn    = "arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/saikat-tooling-tgt/dfe733266106ad8c"
saikatsen@Saikats-MacBook-Pro PBL17 % terraform plan
data.aws_availability_zones.available: Reading...
data.aws_route53_zone.saikat: Reading...
aws_sns_topic.saikat-sns: Refreshing state... [id=arn:aws:sns:us-east-1:811613581700:Default_CloudWatch_Alarms_Topic]
aws_kms_key.ACS-kms: Refreshing state... [id=b9a0fce6-c0fc-4a97-9797-c534509cea04]
aws_acm_certificate.saikat: Refreshing state... [id=arn:aws:acm:us-east-1:811613581700:certificate/4a30b558-914c-4440-98a6-ab08c7bc4896]
aws_vpc.main: Refreshing state... [id=vpc-0c4013f17b37a5213]
aws_iam_policy.policy: Refreshing state... [id=arn:aws:iam::811613581700:policy/ec2_instance_policy]
aws_iam_role.ec2_instance_role: Refreshing state... [id=ec2_instance_role]
data.aws_availability_zones.available: Read complete after 0s [id=us-east-1]
random_shuffle.az_list: Refreshing state... [id=-]
data.aws_route53_zone.saikat: Read complete after 2s [id=Z031762920PPGKBQDT844]
aws_route53_record.saikat["*.saikat-devops.click"]: Refreshing state... [id=Z031762920PPGKBQDT844__0bd2331c8182b3f520abc0aa54564b49.saikat-devops.click._CNAME]
aws_iam_role_policy_attachment.test-attach: Refreshing state... [id=ec2_instance_role-20230117014752970100000002]
aws_iam_instance_profile.ip: Refreshing state... [id=aws_instance_profile_test]
aws_kms_alias.alias: Refreshing state... [id=alias/kms]
aws_efs_file_system.ACS-efs: Refreshing state... [id=fs-0cccbe2987395f177]
aws_acm_certificate_validation.saikat: Refreshing state... [id=2023-01-17 01:48:15.375 +0000 UTC]
aws_route_table.private-rtb: Refreshing state... [id=rtb-05f0f8c1077eb3db5]
aws_security_group.datalayer-sg: Refreshing state... [id=sg-04f05f281781252c1]
aws_lb_target_group.tooling-tgt: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/saikat-tooling-tgt/dfe733266106ad8c]
aws_subnet.public[1]: Refreshing state... [id=subnet-039b237d2b39f0ee7]
aws_subnet.public[0]: Refreshing state... [id=subnet-0a8c1e9ff8151ed42]
aws_security_group.int-alb-sg: Refreshing state... [id=sg-0a45b7b95e295041d]
aws_lb_target_group.nginx-tgt: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/nginx-tgt/68c93702c4ee59ef]
aws_security_group.ext-alb-sg: Refreshing state... [id=sg-09c20452387bc6a46]
aws_security_group.webserver-sg: Refreshing state... [id=sg-08a7f6722841d0660]
aws_route_table.public-rtb: Refreshing state... [id=rtb-040778f31bf74ac86]
aws_internet_gateway.ig: Refreshing state... [id=igw-0bebf379fc11e7089]
aws_security_group.nginx-sg: Refreshing state... [id=sg-042d25e10d2e9204d]
aws_subnet.private[2]: Refreshing state... [id=subnet-09caf8712b32dc16e]
aws_security_group.bastion_sg: Refreshing state... [id=sg-030075847036d548f]
aws_subnet.private[1]: Refreshing state... [id=subnet-0e4f4ef045734903f]
aws_lb_target_group.wordpress-tgt: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/wordpress-tgt/33f4fcd5bfc27e52]
aws_subnet.private[0]: Refreshing state... [id=subnet-0bb4cef51b048e914]
aws_subnet.private[3]: Refreshing state... [id=subnet-0bb4717832bff519e]
aws_efs_access_point.tooling: Refreshing state... [id=fsap-0c57dbaadaa473183]
aws_lb.ext-alb: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:loadbalancer/app/ext-alb/e1e8d6e3e203fe45]
aws_efs_access_point.wordpress: Refreshing state... [id=fsap-05c2f42e7607c5b18]
aws_launch_template.tooling-launch-template: Refreshing state... [id=lt-0f4ea7509e95088b3]
aws_security_group_rule.inbound-nfs-port: Refreshing state... [id=sgrule-3986560262]
aws_launch_template.wordpress-launch-template: Refreshing state... [id=lt-01c570df5ee40218d]
aws_security_group_rule.inbound-web-https: Refreshing state... [id=sgrule-1091876913]
aws_security_group_rule.inbound-mysql-webserver: Refreshing state... [id=sgrule-2119730572]
aws_route_table_association.public-subnets-assoc[0]: Refreshing state... [id=rtbassoc-04164cc105cffd9bc]
aws_route_table_association.public-subnets-assoc[1]: Refreshing state... [id=rtbassoc-0f7caee9f51c6bc6f]
aws_eip.nat_eip: Refreshing state... [id=eipalloc-052e1b8366c110276]
aws_route.public-rtb-route: Refreshing state... [id=r-rtb-040778f31bf74ac861080289494]
aws_security_group_rule.inbound-nginx-http: Refreshing state... [id=sgrule-4143344385]
aws_security_group_rule.inbound-ialb-https: Refreshing state... [id=sgrule-3710512551]
aws_launch_template.nginx-launch-template: Refreshing state... [id=lt-02070bc11a825c366]
aws_security_group_rule.inbound-bastion-ssh: Refreshing state... [id=sgrule-3488766914]
aws_security_group_rule.inbound-mysql-bastion: Refreshing state... [id=sgrule-2655927309]
aws_launch_template.bastion-launch-template: Refreshing state... [id=lt-0ae167430fcc0dad7]
aws_security_group_rule.inbound-web-ssh: Refreshing state... [id=sgrule-2958450141]
aws_db_subnet_group.ACS-rds: Refreshing state... [id=acs-rds]
aws_lb.ialb: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:loadbalancer/app/ialb/3b6916006dd2283c]
aws_efs_mount_target.subnet-1: Refreshing state... [id=fsmt-09c9076f2e76b778c]
aws_efs_mount_target.subnet-2: Refreshing state... [id=fsmt-0f3ca87dfa14bdd08]
aws_route_table_association.private-subnets-assoc[0]: Refreshing state... [id=rtbassoc-06c43cd86938cf1ed]
aws_route_table_association.private-subnets-assoc[3]: Refreshing state... [id=rtbassoc-031924301c8778bda]
aws_route_table_association.private-subnets-assoc[1]: Refreshing state... [id=rtbassoc-0eb33900620535f83]
aws_route_table_association.private-subnets-assoc[2]: Refreshing state... [id=rtbassoc-09d680545f0c0de0c]
aws_autoscaling_group.wordpress-asg: Refreshing state... [id=wordpress-asg]
aws_nat_gateway.nat: Refreshing state... [id=nat-0fcba80facdd222bd]
aws_autoscaling_group.tooling-asg: Refreshing state... [id=tooling-asg]
aws_lb_listener.nginx-listner: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:listener/app/ext-alb/e1e8d6e3e203fe45/1d4ee547b3a7df36]
aws_route53_record.tooling: Refreshing state... [id=Z031762920PPGKBQDT844_tooling.saikat-devops.click_A]
aws_route53_record.wordpress: Refreshing state... [id=Z031762920PPGKBQDT844_wordpress.saikat-devops.click_A]
aws_autoscaling_group.nginx-asg: Refreshing state... [id=nginx-asg]
aws_autoscaling_group.bastion-asg: Refreshing state... [id=bastion-asg]
aws_lb_listener.web-listener: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:listener/app/ialb/3b6916006dd2283c/07f12dd7a030af94]
aws_route.private-rtb-route: Refreshing state... [id=r-rtb-05f0f8c1077eb3db51080289494]
aws_db_instance.ACS-rds: Refreshing state... [id=terraform-2023011701482502350000000b]
aws_lb_listener_rule.tooling-listener: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:811613581700:listener-rule/app/ialb/3b6916006dd2283c/07f12dd7a030af94/29e022b6d8681367]
aws_autoscaling_attachment.asg_attachment_wordpress: Refreshing state... [id=wordpress-asg-20230117021237629100000006]
aws_autoscaling_attachment.asg_attachment_nginx: Refreshing state... [id=nginx-asg-20230117021310982700000007]
aws_autoscaling_attachment.asg_attachment_tooling: Refreshing state... [id=tooling-asg-20230117021237609700000005]
aws_autoscaling_notification.saikat_notifications: Refreshing state... [id=arn:aws:sns:us-east-1:811613581700:Default_CloudWatch_Alarms_Topic]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create
  ~ update in-place

Terraform will perform the following actions:

  # aws_autoscaling_attachment.asg_attachment_nginx will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
      + autoscaling_group_name = "nginx-asg"
      + id                     = (known after apply)
      + lb_target_group_arn    = "arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/nginx-tgt/68c93702c4ee59ef"
    }

  # aws_autoscaling_attachment.asg_attachment_tooling will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
      + autoscaling_group_name = "tooling-asg"
      + id                     = (known after apply)
      + lb_target_group_arn    = "arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/saikat-tooling-tgt/dfe733266106ad8c"
    }

  # aws_autoscaling_attachment.asg_attachment_wordpress will be created
  + resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
      + autoscaling_group_name = "wordpress-asg"
      + id                     = (known after apply)
      + lb_target_group_arn    = "arn:aws:elasticloadbalancing:us-east-1:811613581700:targetgroup/wordpress-tgt/33f4fcd5bfc27e52"
    }

  # aws_route.private-rtb-route will be updated in-place
  ~ resource "aws_route" "private-rtb-route" {
      + gateway_id             = "nat-0fcba80facdd222bd"
        id                     = "r-rtb-05f0f8c1077eb3db51080289494"
      - nat_gateway_id         = "nat-0fcba80facdd222bd" -> null
        # (4 unchanged attributes hidden)
    }

Plan: 3 to add, 1 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

```
