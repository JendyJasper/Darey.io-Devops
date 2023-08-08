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


