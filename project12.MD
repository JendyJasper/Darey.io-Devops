# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)


## Step 1 – Jenkins job enhancement

> We will update our Jenkins job so it doesn't keep creating mew directory for every push we make to github, which becomes difficult to manager and consumes space also. We will use the `copy Artifact` plugin for this.

- Go to your `Jenkins-Ansible` server and create a new directory called `ansible-config-artifact` – we will store there all artifacts after each build.
- Change permissions to this directory, so Jenkins could save files there – `chmod -R 0777 /home/ubuntu/ansible-config-artifact`
- Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins
- Create a new Freestyle project and name it `save_artifacts`.
- This project will be triggered by the completion of the existing `ansible` project.
__________________________________________________________________

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a6f2a4ce-cfa6-4269-a11c-a7a1d5ce9495">

__________________________________________________________________

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a7bbee1b-8a5e-45b3-94e7-3b32666ffec6">

- The main idea of save_artifacts project is to save artifacts into `/home/ubuntu/ansible-config-artifact directory`. 
- To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory.

__________________________________________________________________

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/0351f4c2-4aa6-4bce-a26c-804e131fd382">

__________________________________________________________________

- Test your set-up by making some changes in README.MD file inside your ansible-config-mgt repository (right inside master branch).

> If both Jenkins jobs have completed one after another,  you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/70ce4901-c37b-41ce-a0a5-7cb9c7383bc6">

________________________________________________________

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/52a7e5b4-592f-42ec-a0a4-f3d2093ceeb6">

__________________________________________________________________

# REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
## Step 2 – Refactor Ansible code by importing other playbooks into site.yml

- Create a new branch, and name it `refactor`.

> Let see code re-use in action by importing other playbooks.

-  Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously.




