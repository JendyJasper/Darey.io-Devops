# Automate Infrastructure Using Terraform Part 1 

> In this project, we are trying to automate the AWS infrastructure provisioning.

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/eaf4bda1-68a9-4cb9-af31-999eb47549a8)
```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```
-----
The below is the s3 bucket created on aws
-----
<img width="1041" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/1a5da328-191b-40c4-acec-87d8c410f848">

### Basic Infrastructure Automation(VPC | Subnets)

Let us create a directory structure:
-    Create a folder called `PBL`
-    Create a file in the folder, name it `main.tf`
It looks like this:
-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/1fffbccf-425c-496f-aa49-f8ff28fe737c">
-----

### Provider and VPC resource section

-    Add AWS as a provider, and a resource to create a VPC in the main.tf file.
-    Provider block informs Terraform that we intend to build infrastructure within AWS.
-    Resource block will create a VPC.

```
provider "aws" {
  region = "eu-central-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```

-    The next thing we need to do, is to download necessary plugins for Terraform to work. Providers and provisioners use these plugins. At this stage, we only have a provider in our main.tf file. So, Terraform will download plugin for AWS provider.
-    Let's accomplish this with `terraform init` command as seen in the below demonstration.

-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/54643f1f-be14-4a72-af56-e31d906f313f">
-----
A new directory has been created after running the terraform init command. the folder is .terraform\. it is where Terraform keep plugins.

> Moving on, let us create the only resource we just defined. aws_vpc. But before we do that, we should check to see what terraform intends to create before we tell it to go ahead and create it.
-    Run `terraform plan`
-`Then, if you are happy with changes planned, execute `terraform apply`
-----
Terraform plan command
-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/4b8caac8-ffff-44bf-88aa-a47d54a4d810">
------

Terraform apply command
-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/81a2c187-755b-464e-8d88-a165357fa98f">
-----

Verify on aws
-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/6be4663f-4c68-4b59-ac9b-e8048c417db9">

> Observations:

    -    A new file is created terraform.tfstate This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
    -    If you also observed closely, you would realize that another file gets created during planning and apply. But this file gets deleted immediately. terraform.tfstate.lock.info This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.
It looks like this:
```
{"ID":"08887bf0-8171-6457-9f43-9375d2070577","Operation":"OperationTypeApply",
"Info":"","Who":"ec2-user@ip-10-0-1-173.ec2.internal",
"Version":"1.5.4","Created":"2023-08-08T10:22:32.01334311Z","Path":"terraform.tfstate"}
```
It is a json format file that stores information about a user: user’s ID, what operation he/she is doing, timestamp, and location of the state file.

### Subnets resource section

According to our architectural design, we require 6 subnets:

-    2 public
-    2 private for webservers
-    2 private for data layer
Let us create the first 2 public subnets.

Add the below configuration to the main.tf file:
```

# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}
```
-    We are creating 2 subnets, therefore declaring 2 resource blocks – one for each of the subnets.
-    We are using the vpc_id argument to interpolate the value of the VPC id by setting it to aws_vpc.main.id. This way, Terraform knows inside which VPC to create the subnet.
-    Run terraform plan and terraform apply
-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/76c029df-94ef-412b-9ab2-023cdba71187">
-----

**Observations:**

-    Hard coded values: Remember our best practice hint from the beginning? Both the availability_zone and cidr_block arguments are hard coded. We should always endeavour to make our work dynamic.
-    Multiple Resource Blocks: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a count argument.

Now let us improve our code by refactoring it.

_First, destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, DO NOT DESTROY an infrastructure that has been deployed to production._

_To destroy whatever has been created run terraform destroy command, and type yes after evaluating the plan._

__result of the destruction__
-----
<img width="1418" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d54e1ea1-61aa-4745-a431-bae9ced0e2f7">


### Fixing The Problems By Code Refactoring
-    Fixing Hard Coded Values: We will introduce variables, and remove hard coding.
    -    Starting with the provider block, declare a variable named region, give it a default value, and update the provider section by referring to the declared variable.
     ```
            variable "region" {
        default = "eu-central-1"
    }

    provider "aws" {
        region = var.region
    }

     ```
