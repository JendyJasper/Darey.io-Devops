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

#### Let us see this in action.
-  Create a new git branch and name it feature/jenkinspipeline-stages `git checkout -b feature/jenkinspipeline-stages`
-  Currently we only have the Build stage. Let us add another stage called `Test`. Paste the code snippet below and push the new changes to GitHub:

```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```

- To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.
  - Click on the "Administration" button
  - Navigate to the Ansible project and click on "Scan repository now"
  - Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.
  - In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/43cca7e8-d844-425f-95e1-498e85fc7379">

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2a8c245a-e6ff-466d-99df-db0d221f3ce5">

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/a216c1d8-936a-4086-a1f4-77c19569a31e">


-----

Add more stages and test nested staged:

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/141383d0-d432-4899-929e-81d720165df4">

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f134f923-8a5f-483d-9d5f-4300b8473519">

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/30e524dc-637b-45bc-9c13-acc452619a52">

-----
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e81c76eb-ad8e-4555-a4d5-3d8e8ab42ab5">


# RUNNING ANSIBLE PLAYBOOK FROM JENKINS
- Installing Ansible on Jenkins
- Installing Ansible plugin in Jenkins UI
- Creating Jenkinsfile from scratch.
- - Note: Ensure that Ansible runs against the Dev environment successfully.
 
Tutorial to guide through the step: 
  -  https://www.youtube.com/watch?v=PRpEbFZi7nI

Tutorial on configuring nginx and php on RHEL/CENTOS server: 
  -  https://www.linuxhelp.com/how-to-test-the-php-configuration-on-apache-and-nginx-web-servers-in-centos-7-6
  -  https://www.youtube.com/watch?v=gd1y6vFGW9w&t=2

Tutorial to install php 7.4
  -  https://computingforgeeks.com/how-to-install-php-7-4-on-centos-rhel-8/

Commands to dynamically set hostname, domain name and IP address to config files:
  -  server_IP {{ ip_address }};
  -  domain: {{ ansible_nodename }}
  -  hostname: {{ansible_default_ipv4.address}}

Tutorial to redirect www or http to non www and https and vice versa
  -  https://phoenixnap.com/kb/redirect-http-to-https-nginx

Tutorial to set up let's encrypt on Ubuntu with auto cert renewal
  -  https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04

Tutorial to set up let's encrypt / certbot on Rhel server
  -  https://www.cyberithub.com/how-to-install-lets-encrypt-certbot-on-rhel-centos-8/

Tutorial to configure nginx as gateway for multiple servers
  -  https://www.technhit.in/configure-nginx-as-gateway-for-multiple-servers/
  > Note: Only the IP address of the reverse proxy server will be added to the DNS A record of the domain. The rest of the work happens inside the proxy server. Remember to update your hosts file to include the public IP address of the servers and their domains.

Tutorial to Install Apache, MySQL, PHP (LAMP) on CentOS/RHEL 7
  -  https://tecadmin.net/install-lamp-apache-mysql-and-php-on-centos-rhel-7/
  >  Remember that php-mysql is for ubuntu while php-mysqlnd is for redhat. Also, when you encounter an error, always try to read the log files of the server to get the actuall reason for the error. The file is usually in /var/log/nginx/error.log.

Tutorial on fixing proxy 502 bad gateway error rhel and maybe ubuntu. Also consider using public IP address for the proxy_pass if private IP still gives the connection to upstream refused error.
  -  https://stackoverflow.com/questions/49597942/load-balancer-on-nginx-give-502-bad-gateway
  -  https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx
  -  https://stackoverflow.com/questions/70111791/nginx-13-permission-denied-while-connecting-to-upstream
  -  https://serverfault.com/questions/819423/reverse-proxy-nginx-bad-gateway




