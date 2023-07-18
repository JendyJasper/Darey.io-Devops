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





