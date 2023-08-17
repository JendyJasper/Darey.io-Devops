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

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/1d06cdcc-58c7-4e35-89c0-16d9b0fd0850)
-----

-  DynamoDB table which we create has an entry which includes state file status

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/b1272d3a-683a-47ed-88d6-5a1d58d4d21d)
-----

Navigate to the DynamoDB table inside AWS and leave the page open in your browser. Run terraform plan and while that is running, refresh the browser and see how the lock is being handled:

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/364babac-2a57-43fc-869a-67c8b277f3b2)
-----

-  After terraform plan completes, refresh DynamoDB table.
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f58daca7-0796-4cad-bc28-f01a6fd2e46c)
-----

-  Add Terraform Output

> Before you run terraform apply let us add an output so that the S3 bucket Amazon Resource Names ARN and DynamoDB table name can be displayed.

*  Create a new file and name it output.tf and add below code.
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
-  Now we have everything ready to go!
-  Let us run terraform apply

>  Terraform will automatically read the latest state from the S3 bucket to determine the current state of the infrastructure. Even if another engineer has applied changes, the state file will always be up to date.

>  Now, head over to the S3 console again, refresh the page, and click the grey “Show” button next to “Versions.” You should now see several versions of your terraform.tfstate file in the S3 bucket:

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/fc7abd0b-7679-4851-bf50-d10512a38ea4)

> With help of remote backend and locking configuration that we have just configured, collaboration is no longer a problem.

> However, there is still one more problem: Isolation Of Environments. Most likely we will need to create resources for different environments, such as: Dev, sit, uat, preprod, prod, etc.

> This separation of environments can be achieved using one of two methods:

- Terraform Workspaces
- Directory-based separation using terraform.tfvars file

### Security Groups refactoring with dynamic block
### For repetitive blocks of code you can use dynamic blocks in Terraform
> Remember, every piece of work you do, always try to make it dynamic to accommodate future changes. Amazon Machine Image (AMI) is a regional service which means it is only available in the region it was created. But what if we change the region later, and want to dynamically pick up AMI IDs based on the available AMIs in that region? This is where we will introduce Map and Lookup functions.

-  Map uses a key and value pairs as a data structure that can be set as a default type for variables.
```
variable "images" {
    type = "map"
    default = {
        us-east-1 = "image-1234"
        us-west-2 = "image-23834"
    }
}
```
>  To select an appropriate AMI per region, we will use a lookup function which has following syntax: `lookup(map, key, [default])`.

>  Note: A default value is better to be used to avoid failure whenever the map data has no key.

```
resource "aws_instace" "web" {
    ami  = "${lookup(var.images, var.region), "ami-12323"}
}
```
_Now, the lookup function will load the variable images using the first parameter. But it also needs to know which of the key-value pairs to use. That is where the second parameter comes in. The key us-east-1 could be specified, but then we will not be doing anything dynamic there, but if we specify the variable for region, it simply resolves to one of the keys. That is why we have used var.region in the second parameter._

### Conditional Expressions

-  If you want to make some decision and choose some resource based on a condition – you shall use Terraform Conditional Expressions.
-  In general, the syntax is as following: condition ? true_val : false_val
```
resource "aws_db_instance" "read_replica" {
  count               = var.create_read_replica == true ? 1 : 0
  replicate_source_db = aws_db_instance.this.id
}
```
-  `true` #condition equals to ‘if true’
-  `?` #means, set to ‘1`
-  `:` #means, otherwise, set to ‘0

### Terraform Modules and best practices to structure your .tf codes
> Modules serve as containers that allow to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). One root module can call other child modules and insert their configurations when applying Terraform config. 

>  You can refer to existing child modules from your root module by specifying them as a source, like this:
```
module "network" {
  source = "./modules/network"
}
```
-  Note that the path to ‘network’ module is set as relative to your working directory.

-  Or you can also directly access resource outputs from the modules, like this:
```
resource "aws_elb" "example" {
  # ...

  instances = module.servers.instance_ids
}
```

### REFACTOR YOUR PROJECT USING MODULES
-  Break down your Terraform codes to have all resources in their respective modules. Combine resources of a similar type into directories within a ‘modules’ directory, for example, like this:
```
- modules
  - ALB: For Apllication Load balancer and similar resources
  - EFS: For Elastic file system resources
  - RDS: For Databases resources
  - Autoscaling: For Autosacling and launch template resources
  - compute: For EC2 and rlated resources
  - VPC: For VPC and netowrking resources such as subnets, roles, e.t.c.
  - security: for creating security group resources
```
- Each module shall contain following files:
```
- main.tf (or %resource_name%.tf) file(s) with resources blocks
- outputs.tf (optional, if you need to refer outputs from any of these resources in your root module)
- variables.tf (as we learned before - it is a good practice not to hard code the values and use variables)
```
**It is also recommended to configure providers and backends sections in separate files but should be placed 
in the root module.**

-  Import module as a `source` and have access to its variables via `var` keyword:
```
module "VPC" {
  source = "./modules/VPC"
  region = var.region
  ...
```

- Refer to a module’s output by specifying the full path to the output variable by using `module.%module_name%.%output_name%` construction:

`subnets-compute = module.network.public_subnets-1`

#### the resulting configuration structure in your working directory may look like this:

-----
<img width="442" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/6a021560-afcb-4974-985a-635044b046ca">

-----
### CREATED RESOURCES BELOW

-----
EC2
-----

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/b416bf31-d5ee-4843-b511-8762377a8e95)

-----
EFS
-----

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ae72554d-55f5-4fdb-ac3a-2169eb8df99a)

-----
VPC and SUBNETS

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/58f87c38-2df4-48a3-8ca6-4677a3942ec4)

-----

NAT GATEWAY
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a4d954cf-7108-44bb-ad40-821cb62c30d6)

-----

ASG
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/9fbd9a27-7de7-46d9-99b5-b7ca900683eb)

-----
Target Groups
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/7d4c0da4-72be-45d1-afb8-e53c865d3101)

-----
RDS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/09a2ea3f-9db1-45ac-8b25-5cdb48bee605)

and a few other reources

https://github.com/JendyJasper/terraform-prov-mgt/tree/project-18
