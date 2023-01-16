
#### INTRODUCTION
--------------------------------------

In continuation to project 16, the remaining resources are created in this project in order to set up a secured infrastructure with Terraform
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
