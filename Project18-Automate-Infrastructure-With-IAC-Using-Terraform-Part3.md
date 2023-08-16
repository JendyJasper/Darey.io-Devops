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
