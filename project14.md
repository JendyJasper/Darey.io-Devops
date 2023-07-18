# Continuous integration with jenkins | ansible | artifactory | sonarqube | php

> In this project, we are going to work on the tooling app and a todo app. The inventory directory looks like this:

<img width="271" alt="Screenshot 2023-07-18 at 3 56 08 PM" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/efd38540-6edc-4281-abb7-ebef8dfe22a0">


- The CI inventory file will look like this:
```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
```


- The dev inventory file will look like this:
```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

- The pentest inventory file will look like this:
```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```

#### NOTES:
> in the pentest inventory file, we have introduced a new concept pentest:children This is because, we want to have a group called pentest which covers Ansible execution against both pentest-todo and pentest-tooling simultaneously. But at the same time, we want the flexibility to run specific Ansible tasks against an individual group.

> The db group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on Ubuntu (in this case user is ubuntu). Therefore, the user required for connectivity and path to python interpreter are different. If all your environment is based on Ubuntu, you may not need this kind of set up.


_______________________________________________________________
> The other environments - `pre-prod, prod, sit and uat` all will have same inventory structure as `dev` environment.

![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2a5bc192-b5bd-4572-a00c-734a80fb278c)

_______________________________________________________________
- CI ENVIRONMENT
  - ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/505ffb9a-6a6d-41fa-925d-d2c8bc542cbf)
 
_______________________________________________________________

- OTHER ENVIRONMENTS
  - ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e3935c5e-fc4e-4af2-9b21-72ad93670683)
 
_______________________________________________________________

  Server                                |      Domain
  :------------------------------------ |:----------------------------------------------------  |                          	
| Jenkins                               |      https://ci.infradev.jendyjasper.com              |
| Sonarqube	                            |      https://sonar.infradev.jendyjasper.com           |
| Artifactory	                          |      https://artifacts.infradev..jendyjasper.com      |
| Production Tooling	                  |      https://tooling.jendyjasper.com                  |
| Pre-Prod Tooling	                    |      https://tooling.preprod.jendyjasper.com          |
| Pentest Tooling	                      |      https://tooling.pentest.jendyjasper.com          |
| UAT Tooling	                          |      https://tooling.uat.jendyjasper.com              |
| SIT Tooling	                          |      https://tooling.sit.jendyjasper.com              | 
| Dev Tooling	                          |      https://tooling.dev.jendyjasper.com              |
| Production TODO-WebApp	              |      https://todo.jendyjasper.com                     |
| Pre-Prod TODO-WebApp	                |      https://todo.preprod.jendyjasper.com             |
| Pentest TODO-WebApp	                  |      https://todo.pentest.jendyjasper.com             |
| UAT TODO-WebApp	                      |      https://todo.uat.jendyjasper.com                 |
| SIT TODO-WebApp	                      |      https://todo.sit.jendyjasper.com                 |
| Dev TODO-WebApp	                      |      https://todo.dev.jendyjasper.com                 |


______________________________________________________________

# ANSIBLE ROLES FOR CI ENVIRONMENT
Add two more roles to ansible:
- SonarQube: Inspection of code quality
- Artifactory: Binary repository or source code repository 
  
`sudo ansible-galaxy init artifactory` and `sudo ansible-galaxy init sonarqube`

- The new role file structure will look like this:

  <img width="289" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/b6a0be99-ab4c-4aaa-9430-abc3df1ddf7b">


## Configuring Ansible For Jenkins Deployment - We will start running Ansible from Jenkins UI.

### Set up Ocean blue Jenkins plugin and connect your Github repo
- Navigate to Jenkins URL
- Install & Open Blue Ocean Jenkins Plugin
- Create a new pipeline
- Select Github
- Connect Jenkins with GitHub using your access token
- Select the repository you want to use and Create a new Pipeline
- Click on Administration to exit the Blue Ocean console because you will create `Jenkinsfile` manually later.
- Here is our newly created pipeline. It takes the name of your GitHub repository.

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/65a216b4-b879-4a65-84eb-acd65e6ea08b">

-----


### Let us create our Jenkinsfile

- Inside the Ansible project, create a new directory `deploy` and start a new file `Jenkinsfile` inside the directory.
- This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage
- Now go back into the Ansible pipeline in Jenkins, and select configure
- Scroll down to `Build Configuration` section and specify the location of the Jenkinsfile at `deploy/Jenkinsfile`
  -----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/bafee983-5eb9-466a-b98b-336a986f94c3">

- Back to the pipeline again, this time click "Build now"
- Try triggering the build again from Blue Ocean interface

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ea1a5ef9-f19c-4bdd-a75e-9e18fe3e4d54">

  - Click on Blue Ocean
  - select your project
  - Click on the play button against the branch

> Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.




