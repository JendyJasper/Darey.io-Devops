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

### NAT Gateways
_Create 1 NAT Gateways and 1 Elastic IP (EIP) addresses_

_Now use similar approach to create the NAT Gateways in a new file called natgateway.tf._

_Note: We need to create an Elastic IP for the NAT Gateway, and you can see the use of depends_on to indicate that the Internet Gateway resource must be available before this should be created. Although Terraform does a good job to manage dependencies, but in some cases, it is good to be explicit._

```
resource "aws_eip" "nat_eip" {
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.nat_eip_tags)
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
      Name = format("%s-Nat", var.nat_gateway_tags)
    },
  )
}
```
> i have set the variables for both in the variable file
-----
<img width="1144" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d9d03e24-267c-4840-acca-94b808c1a391">
-----
<img width="1144" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/783a33c0-931f-45b8-bfa8-85e76395eb73">

### AWS ROUTES
_Create a file called route_tables.tf and use it to create routes for both public and private subnets, create the below resources. Ensure they are properly tagged._

  -  aws_route_table
  -  aws_route
  -  aws_route_table_association

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
> Now if you run terraform plan and terraform apply it will add the following resources to AWS in multi-az set up:
* Our main vpc
* 2 Public subnets
* 4 Private subnets
* 1 Internet Gateway
* 1 NAT Gateway
* 1 EIP
* 2 Route tables
-----
<img width="1144" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/c43d52f0-6d71-4234-9c3b-d0a190579db1">


__Now, we are done with Networking part of AWS set up, let us move on to Compute and Access Control configuration automation using Terraform!__


### AWS Identity and Access Management
> We want to pass an IAM role our EC2 instances to give them access to some specific resources, so we need to do the following:

-  Create AssumeRole
_Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access_

-  Add the following code to a new file named roles.tf
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
> In this code we are creating AssumeRole with AssumeRole policy. It grants to an entity, in our case it is an EC2, permissions to assume the role.

-  Create IAM policy for this role
_This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform action describe applied to EC2 instances_
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

- Attach the Policy to the IAM Role
_This is where, we will be attaching the policy which we created above, to the role we created in the first step._
```
resource "aws_iam_role_policy_attachment" "test-attach" {
        role       = aws_iam_role.ec2_instance_role.name
        policy_arn = aws_iam_policy.policy.arn
    }
```

-  Create an Instance Profile and interpolate the IAM Role
```
   resource "aws_iam_instance_profile" "ip" {
        name = "aws_instance_profile_test"
        role =  aws_iam_role.ec2_instance_role.name
    }
```
**We are pretty much done with Identity and Management part for now, let us move on and create other resources required.**

### Resources to be created
**As per our architecture we need to do the following:**

*  Create Security Groups
*  Create Target Group for Nginx, WordPress and Tooling
*  Create certificate from AWS certificate manager
*  Create an External Application Load Balancer and Internal Application Load Balancer.
*  create launch template for Bastion, Tooling, Nginx and WordPress
*  Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
*  Create Elastic Filesystem
*  Create Relational Database (RDS)

_Let us create some Terraform configuration code to accomplish these tasks._
-----
<img width="1419" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/03fbc466-2e92-4ba2-afc2-2314233af8fb">

-----

<img width="1419" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ca35f0d3-a247-443d-a77b-ecce9ff814e1">

-----

### CREATE SECURITY GROUPS
_We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it._
-  Create a file and name it security.tf, copy and paste the code below
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
# We used the aws_security_group_rule to reference another security group in a security group.
-----
<img width="1419" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/776ed56c-6d4d-4934-8c8e-128259b50910">


-----

### CREATE CERTIFICATE FROM AMAZON CERIFICATE MANAGER
- Create cert.tf file and add the following code snippets to it.
```
# The entire section create a certiface, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in jendyjasper.com
resource "aws_acm_certificate" "jendyjasper" {
  domain_name       = "*.jendyjasper.com"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "jendyjasper" {
  name         = "jendyjasper.com"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "jendyjasper" {
  for_each = {
    for dvo in aws_acm_certificate.jendyjasper.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.jendyjasper.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "jendyjasper" {
  certificate_arn         = aws_acm_certificate.jendyjasper.arn
  validation_record_fqdns = [for record in aws_route53_record.jendyjasper : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.jendyjasper.zone_id
  name    = "tooling.jendyjasper.com"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.jendyjasper.zone_id
  name    = "wordpress.jendyjasper.com"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```

### Create an external (Internet facing) Application Load Balancer (ALB)
-  Create a file called alb.tf
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

-  To inform our ALB  where  to route the traffic we need to create a Target Group to point to its targets:
```
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
-  Then we will need to create a Listner for this target Group

```
resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.jendyjasper.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}
```

-  Add the following outputs to output.tf to print them on screen
```
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}
```

### Create an Internal (Internal) Application Load Balancer (ALB)
-  For the Internal Load balancer we will fillow thje same concepts with the external load balancer.
```
# ----------------------------
#Internal Load Balancers for webservers
#---------------------------------

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

**To inform our ALB to where route the traffic we need to create a Target Group to point to its targets:**
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

-  Then we will need to create a Listner for this target Group
```
# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes

resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.jendyjasper.certificate_arn

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
      values = ["tooling.jendyjasper.com"]
    }
  }
}
```
