# Continue Infrastructure Automation with Terraform - Continued from where we stopped in project 16

> Note: You can add multiple tags as a default set. for example, in out terraform.tfvars file we can have default tags defined.

```
tags = {
  Enviroment      = "production" 
  Owner-Email     = "jendydevops@gmail.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```

> Update the variables.tf to declare the variable tags used in the format above;

```
variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```

> Now you can tag all you resources using the format below

```
tags = merge(
    var.tags,
    {
      Name = "Name of the resource",
      year = 2023
    },
  )
  ```

**The merge function here combines all the tags you defined in the tfvars file and any other tag you may define inline in the resource**

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/648bbbc7-387d-4999-9e70-43049dbf2b19)
-----

<img width="362" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5bbaa2a3-d912-4e92-a4dd-488185f71c7a">

_The nice thing about this is – anytime we need to make a change to the tags, we simply do that in one single place (terraform.tfvars)._

_But, our key-value pairs are hard coded. So, go ahead and work out a fix for that. Simply create variables for each value and use var.variable_name as the value of each of the keys.
Apply the same best practices for all other resources you will create further._

### Internet Gateways & format() function
__Create an Internet Gateway in a separate Terraform file internet_gateway.tf__
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

_Did you notice how we have used format() function to dynamically generate a unique name for this resource? The first part of the %s takes the interpolated value of aws_vpc.main.id while the second %s appends a literal string IG and finally an exclamation mark is added in the end._

_If any of the resources being created is either using the count function, or creating multiple resources using a loop, then a key-value pair that needs to be unique must be handled differently._

_For example, each of our subnets should have a unique name in the tag section. Without the format() function, we would not be able to see uniqueness. With the format function, each private subnet’s tag will look like this._

```
Name = PrvateSubnet-0
Name = PrvateSubnet-1
Name = PrvateSubnet-2
```

Lets try and see that in action.
```
tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index)
    } 
  )
```
-----
<img width="1144" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8038301f-d519-41da-98ec-d662ab72717c">


> The number started from zero because the index starts at 0. you can tweak it by adding +1 to count.index as shown below:
```
tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index+1)
    } 
  )
```

-----
<img width="1144" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8415dea1-a26f-4be3-9031-3eddc494c7e9">

