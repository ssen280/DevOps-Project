
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
