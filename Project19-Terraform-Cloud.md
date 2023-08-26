# What Terraform Cloud is and why use it

Terraform is an open-source system, that you install and run a Virtual Machine (VM) that you have to create, maintain and keep up to date. In the Cloud world it is quite common to provide a managed version of an open-source software. Managed means that you do not have to install, configure and maintain it yourself – you just create an account and use it “as A Service”.

Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

By default, Terraform CLI performs operation on the server where it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with Terraform – you need a consistent remote environment with remote workflow and shared state to run Terraform commands.

Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called remote operations.

### Migrate your .tf codes to Terraform Cloud
1. Create a Terraform Cloud account
2. ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/4af04256-0b8d-4175-a14f-e750eb5d2adc)
3. Create an organization: Select “Start from scratch”, choose a name for your organization and create it.
4. Configure a workspace. We will use version control workflow as the most common and recommended way to run Terraform commands triggered from our git repository.
5. Create a new repository in your GitHub and call it terraform-cloud, push your Terraform codes developed in the previous projects to the repository.
6. Choose version control workflow and you will be prompted to connect your GitHub account to your workspace – follow the prompt and add your newly created repository to the workspace.
7. ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/07f3dc22-c5e4-4f46-998e-d2000e7e640e)
8. Choose a repository
9. ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a92fdf4c-c35f-46a5-923c-6527a1dbe269)
10. Move on to “Configure settings”, provide a description for your workspace and leave all the rest settings default, click “Create workspace”.
11. Configure variables
12. > Terraform Cloud supports two types of variables: environment variables and Terraform variables. Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.

> Set two environment variables: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, set the values that you used in Project 16. These credentials will be used to provision your AWS infrastructure by Terraform Cloud.

> After you have set these 2 environment variables – your Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

13. ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/87f37052-591b-499a-acf1-6a0f7e1c6bd0)
14. Now it is time to run our Terraform scripts, but in our previous project which was project 18, we talked about using Packer to build our images, and Ansible to configure the infrastructure, so for that we are going to make few changes to our our existing repository from Project 18.
15. Before you proceed ensure you have the following tools installed on your local machine; Packer and Ansible
16. See the refactored code at https://github.com/JendyJasper/terraform-cloud/tree/main
17. Run terraform plan and terraform apply from web console
18. ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/0d5b6421-1a3f-497f-8728-88dcc8d9048e)
> Switch to “Runs” tab and click on “Queue plan manually” button. If planning has been successful, you can proceed and confirm Apply – press “Confirm and apply”, provide a comment and “Confirm plan”
19. Check the logs and verify that everything has run correctly. Note that Terraform Cloud has generated a unique state version that you can open and see the codes applied and the changes made since the last run.
20. ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2b0a0f77-6958-488b-a443-2d6f9ee9b686)
21. Test automated terraform plan
22. By now, you have tried to launch plan and apply manually from Terraform Cloud web console. But since we have an integration with GitHub, the process can be triggered automatically. Try to change something in any of .tf files and look at “Runs” tab again – plan must be launched automatically, but to apply you still need to approve manually. Since provisioning of new Cloud resources might incur significant costs. Even though you can configure “Auto apply”, it is always a good idea to verify your plan results before pushing it to apply to avoid any misconfigurations that can cause ‘bill shock’.

###PACKER

What is Packer?
Packer is a tool that lets you create identical machine images for multiple platforms from a single source template. Packer can create golden images to use in image pipelines. 
When you create and run packer build, you should get similar result to that shown below.
<img width="1431" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a7782972-aa80-4bdb-ba1b-cc65c7207c34">
Verify the AMIs created on AWS.
<img width="1431" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d571df1d-fd26-44ff-90a2-03721e599842">






