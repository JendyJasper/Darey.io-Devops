# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company
that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) 
for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made 
to use a reverse proxy technology from NGINX to achieve this.

> Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, 
ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accommodate to 
increased traffic and, at the same time, has reasonable cost.

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/0e530aef-316f-4847-99a5-6ce368564361)
-----

###  Starting Off Your AWS Cloud Project
There are few requirements that must be met before you begin:

-  Properly configure your AWS account and Organization Unit: https://youtu.be/9PQYCc_20-Q
    -  Create an AWS Master account. (Also known as Root Account)
    -  Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
    -  Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
    -  Move the DevOps account into the Dev OU.
    -  Login to the newly created AWS account using the new email address.
-  Create a domain name for your fictitious company.
-  Create a hosted zone in AWS, and map it to your domain: https://youtu.be/IjcHp94Hq8A

>  NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

        -  Project: `<Give your project a name>`
        -  Environment: `<dev>`
        -  Automated: `<No> `(If you create a resource using an automation tool, it would be `<Yes>`)
