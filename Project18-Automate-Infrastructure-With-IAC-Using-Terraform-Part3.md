# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

### Introducing Backend on S3

> Here is our plan to Re-initialize Terraform to use S3 backend:
-  Add S3 and DynamoDB resource blocks before deleting the local state file
-  Update terraform block to introduce backend and locking
-  Re-initialize terraform
-  Delete the local tfstate file and check the one in S3 bucket
-  Add outputs
-  terraform apply

-  Create a file and name it backend.tf. Add the below code and replace the name of the S3 bucket you created in Project-16.
```
# Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.
resource "aws_s3_bucket" "terraform_state" {
  bucket = "dev-terraform-bucket"
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
```

> Next, we will create a DynamoDB table to handle locks and perform consistency checks. In previous projects,
> locks were handled with a local file as shown in terraform.tfstate.lock.info. Since we now have a team mindset,
> causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore,
> with a cloud storage database like DynamoDB, anyone running Terraform against the same infrastructure can use a central
> location to control a situation where Terraform is running at the same time from multiple different people.

-  Dynamo DB resource for locking and consistency checking:
```
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
> Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run terraform apply to provision resources

-  Configure S3 Backend
```
terraform {
  backend "s3" {
    bucket         = "dev-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

-  Now it's time to re-initialize the backend. Run terraform init and confirm you are happy to change the backend by typing yes

-  Verify the changes
-  Before doing anything if you opened AWS now to see what happened you should be able to see the following:

-  <img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/79646bd6-9801-403b-bbfe-f0bb97f69419">

tfstatefile is now inside the S3 bucket


