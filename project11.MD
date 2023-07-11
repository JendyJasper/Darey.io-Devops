# Automate project 7 to 10 using ansible and jenkins

### INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
- create a new repository and name it ansible-config-mgt.
- Install Ansible `sudo apt update` and `sudo apt install ansible`
- Check your Ansible version by running ansible --version

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/fd4a4c8e-3995-4c09-a89d-d933df7bc238">

- Configure Jenkins build job to save your repository content every time you change it 
  - Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
  - Configure Webhook in GitHub and set webhook to trigger ansible build.
  - Configure a Post-build job to save all (**) files.

- Test your setup by making some change in README.MD file in main branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/55a590fa-cd33-43de-b002-e6eafb0a24eb">


---------------------------------------------------------------------
## BEGIN ANSIBLE DEVELOPMENT

- In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
- Checkout the newly created feature branch to your local machine and start building your code and directory structure: `git checkout -b prj11 `
- Create a directory and name it playbooks – it will be used to store all your playbook files: `mkdir playbooks`
- Create a directory and name it inventory – it will be used to keep your hosts organised: `mkdir inventory`
- Within the playbooks folder, create your first playbook, and name it common.yml: `touch common.yml`
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively: `touch inventory/dev.yml inventory/staging.yml inventory/uat.yml inventory/prod.y
ml`

### Set up an Ansible Inventory
- Save below inventory structure in the inventory/dev file to start configuring your development servers.
> Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:
- On the local machine where you have your pem key downloaded, run `eval 'ssh agent -s'` and also run `ssh-add <path-to=private-key>`
- Confirm the key has been added with the command below, you should see the name of your key; `ssh-add -l`
- Now log into the the Jenkins-Ansible server using ssh-agent: `ssh -A ubuntu@public-ip`
- Test that your Jenkins-Ansible server can connect to other instances that use the same private key:
  - on the Jenkins-Ansible server, run 'ssh-add -l`. You should see your private key there too
  - now try to ssh into other instances from the Jenkins-Ansible server by running `ssh ubuntu@<private-key>/<public-key>` or `ssh ec2-user@<private-key>/<public-key>`

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/52ec8708-279e-4751-a8c5-82225d534b86">

<img width="932" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e01c5312-404c-4c42-8321-ba8fa2fd1fc4">

#### Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

- Update your inventory/dev.yml file with this snippet of code: replace with actual IP addresses
```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

### CREATE A COMMON PLAYBOOK
- Let's give Ansible the instructions on what needs to be done on all the attached servers listed in inventory/dev
- In common.yml playbook, write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure. Update playbooks/common.yml file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

#### Update GIT with the latest code

- use git commands to add, commit and push your branch to GitHub.
```
git status

git add <selected files>

git commit -m "commit message"
```

- Push the changes to the prj11 branch
- Create a pull request
- Review and merge
- Head back to your terminal, checkout from the feature branch into the master, and pull down the latest changes

> Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory` on Jenkins-Ansible server.

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8fe2e044-befd-47ed-a72e-f68d7acf9727">

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2790aaa7-1417-4004-85ea-83a97f3833f5">


### Run first Ansible Test

- Login to the Jenkins-Ansible server
- change directory to the folder that holds that artifacts that are been sent from github webhook: `cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
- ansible-playbook -i inventory/dev.yml playbooks/common.yml

<img width="895" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/cd48cbee-6e7a-4087-b1fe-bba6beabf7fc">

- login to few of the servers to confirm wireshark was installed, time zone shows WAT, the directory and file were created

<img width="1159" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/eff27836-a4bd-4730-8510-b8a933350871">

