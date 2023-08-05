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
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/86c6fd39-89bf-4b17-8bf0-0155e78b1edb)
-----
### ELASTIC IP
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/797687b0-a216-402d-9bb0-253d50f85438)
-----
### SECURITY GROUPS
------
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ada49385-bb97-4202-b387-10da13e64d7d)
### EC2
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/9b8f3d86-55ef-4882-a9a0-4f04df1e8850)
-----
### LAUNCH TEMPLATES
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/42a9c8de-4743-4e25-b39b-977fd717408e)
-----
### LOAD BALANCERS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/c9b88763-4f1a-4d04-b95c-e2fdc90431c3)
-----
### TARGET GROUPS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/567231f7-e4c5-4423-b03d-5ea8c5b92131)
-----
### AUTO SCALING GROUPS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e01a34d2-a700-4f79-bd95-69adbfeeec2e)
-----
### EFS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8736d064-a84e-4414-803b-33f46c7e5112)
-----
### EFS ACCESS POINTS
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/363a98fe-54ae-4f77-9b2a-518d3c9c6de4)
------
### SUBNET GROUPS
------
<img width="1430" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e49814a0-8d34-47c3-adae-96c9e0090ca3">
------
### RDS
------
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/07d5c55b-5995-4f1a-a866-1de893f70f7b)
-----

<img width="1430" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2a9d6ce8-f8ef-42c0-bbe9-e41f86887835">
-----
<img width="1430" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e1d87cca-ad78-42b8-ab23-fd42441c49b7">
