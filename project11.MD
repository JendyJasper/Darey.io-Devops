#Automate project 7 to 10 using ansible and jenkins

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
