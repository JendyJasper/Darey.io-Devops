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
`touch playbooks/site.yml`

- Create a new folder in root of the repository and name it `static-assignments`. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work.
`mkdir static-assignments`

- Move common.yml file into the newly created static-assignments folder.
`mv playbooks/common.yml static-assignments/common.yml`

- Inside site.yml file, import common.yml playbook.
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

- The new directory structure looks like below:

<img width="354" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/973e1c02-fa84-4d9b-9fd8-fe69adbf7a42">

- Run `ansible-playbook` command against the dev environment

> Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure the deletion of Wireshark utility:
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

- update site.yml with `- import_playbook: ../static-assignments/common-del.yml` instead of common.yml and run it against dev servers:
```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
```
- Then run it against dev servers

```
cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```


# CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’

## Step 3 – Configure UAT Webservers with a role ‘Webserver’

> We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.


- Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.
- To create a role, you must create a directory called `roles/`, relative to the playbook file or in `/etc/ansible/` directory.
- There are two ways how you can create this folder structure:
  - Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)
   ```
    mkdir roles
    cd roles
    ansible-galaxy init webserver
  ```

