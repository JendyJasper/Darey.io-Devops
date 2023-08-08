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

