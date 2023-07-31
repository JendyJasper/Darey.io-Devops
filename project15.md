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

    -  Project: <Give your project a name>
    -  Environment: <dev>
    -  Automated: <No> (If you create a resource using an automation tool, it would be <Yes>)

-----
  ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/6af05c9c-5a10-44c9-a33a-244ba51fab9e)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/90ab3599-f306-4e8d-9372-63feb38132c3)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/bb715578-f6d1-46ac-b2d7-efd5dc499d27)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/4083bb45-7219-42a5-b3ac-ffbc099f3a44)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d2a05256-5c5c-4a79-87ee-81b799a252d5)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e305b288-5b5a-4bb4-921f-9df52b6998f3)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/1319f087-74ff-4faa-95fb-08109777e198)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/02790d8e-c0a7-48ad-b6bc-b1424c56c1e5)
------
# THE AWS PAGE
-----
### VPC 
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/088c18ce-3647-47ba-900b-ed05c72c6d41)
-----
### SUBNETS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/c75a7359-f575-427d-b46a-4aa253287820)
-----
### ROUTE TABLES
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/c5d8980e-6c27-4eea-8c61-0816d9fb9e1c)
-----
### INTERNET GATEWAYS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/276236aa-131e-481b-a2ec-56b962f24ecc)
-----
#### NAT GATEWAYS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ffc6592c-00f9-4d12-98b5-6403c56a9fe5)
-----
### ELASTIC IP
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/797687b0-a216-402d-9bb0-253d50f85438)
-----

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/86c6fd39-89bf-4b17-8bf0-0155e78b1edb)



