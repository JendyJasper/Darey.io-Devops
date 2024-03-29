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

Tutorial to enable your server to connect to a database over HTTP(Especially for RHEL server)
  -  https://stackoverflow.com/questions/41178774/connect-database-error-type-2002-permission-denied

Tutorial to print the actual database connection error which should help during debugging
  -  https://www.plus2net.com/php_tutorial/mysqli_connection.php


Tutorial to access the environment variables of the machine which ansible is running from / and also to access the Jenkins environment variables if you are runnimg ansible from jenkins
  - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/env_lookup.html#examples
  - https://github.com/jenkinsci/ansible-plugin/blob/main/README.md
  - https://wiki.jenkins.io/display/JENKINS/Building+a+software+project

    - examples code:
    ```
    ---
    - hosts: test
      name: print workspace
      tasks:
        - name: get workspace
          debug:
            msg: "'{{ lookup('ansible.builtin.env', 'WORKSPACE')}}' is the workspace environment variable"
    ```


# CI/CD PIPELINE FOR TODO APPLICATION
### Phase 1 – Prepare Jenkins
-  Fork the repository below into your GitHub account: https://github.com/darey-devops/php-todo.git
-  On you Jenkins server, install PHP, its dependencies and Composer tool  `sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}`
-  Install Jenkins plugins
    -  Plot plugin
    -  Artifactory plugin
      
    >  We will use plot plugin to display tests reports, and code coverage information.
    >  The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.
-  In Jenkins UI configure Artifactory
-  Configure the server ID, URL and Credentials, run Test Connection.

### Phase 2 – Integrate Artifactory repository with Jenkins
-  Create a dummy Jenkinsfile in the repository
-  Using Blue Ocean, create a multibranch Jenkins pipeline
-  In the database server, create database and user
  ```
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```
-  Update the database connectivity requirements in the file .env.sample
-  Update Jenkinsfile with proper pipeline configuration
```
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```
-  The required file by PHP is .env so we are renaming .env.sample to .env
-  Composer is used by PHP to install all the dependent libraries used by the application
-  php artisan uses the .env file to setup the required database objects – (After successful run of this step, login to the database, run -show tables and you will see the tables being created for you)

Update the Jenkinsfile to include Unit tests step
    ```
    stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      }
      ```
###Phase 3 – Code Quality Analysis
For PHP the most commonly tool used for code quality analysis is phploc. The data produced by phploc can be ploted onto graphs in Jenkins.

-  Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.
  ```
stage('Code Analysis') {
  steps {
        sh 'phploc app/ --log-csv build/logs/phploc.csv'

  }
}
```

-  Plot the data using plot Jenkins plugin
```
stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }
```
>  You should now see a Plot menu item on the left menu. Click on it to see the charts.

-  Bundle the application code for into an artifact (archived package) upload to Artifactory
```
stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
```
-  Publish the resulted artifact into Artifactory
```
stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }

        }
```
-  Deploy the application to the dev environment by launching Ansible pipeline
```
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
```

> The build job used in this step tells Jenkins to start another job. In this case it is the ansible-project job, and we are targeting the main branch. Hence, we have ansible-project/main. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. The name of the parameter is env and its value is dev. Meaning, deploy to the Development environment.


# SONARQUBE INSTALLATION
### Install SonarQube on Ubuntu 20.04 With PostgreSQL as Backend Database
> We will make some Linux Kernel configuration changes to ensure optimal performance of the tool – we will increase vm.max_map_count, file discriptor and ulimit.

-  Tune Linux Kernel
-  This can be achieved by making session changes which does not persist beyond the current session terminal.
```
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
-  To make a permanent change, edit the file `/etc/security/limits.conf` and append the below
```
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```

-  Before installing, let us update and upgrade system packages:
```
sudo apt-get update
sudo apt-get upgrade
```
-  Install wget and unzip packages
```
sudo apt-get install wget unzip -y
```
-  Install OpenJDK and Java Runtime Environment (JRE) 11
```
 sudo apt-get install openjdk-11-jdk -y
 sudo apt-get install openjdk-11-jre -y
```
-  Set default JDK – To set default JDK or switch to OpenJDK enter below command: `sudo update-alternatives --config java`
-  If you have multiple versions of Java installed, you should see a list like below:
```
Selection    Path                                            Priority   Status

------------------------------------------------------------

  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode

  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode

  2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

* 3            /usr/lib/jvm/java-8-oracle/jre/bin/java          1081      manual mode
```
Type "1" to switch OpenJDK 11

Verify the set JAVA Version: `java -version`

### Install and Setup PostgreSQL 10 Database for SonarQube

-  The command below will add PostgreSQL repo to the repo list:

`sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'`

-  Download PostgreSQL software
`wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -`
-  Install PostgreSQL Database Server
`sudo apt-get -y install postgresql postgresql-contrib`
-  Start PostgreSQL Database Server
`sudo systemctl start postgresql`
-  Enable it to start automatically at boot time
`sudo systemctl enable postgresql`
-  Change the password for default postgres user (Pass in the password you intend to use, and remember to save it somewhere)
`sudo passwd postgres`
-  Switch to the postgres user
`su - postgres`
-  Create a new user by typing
`createuser sonar`
-  Switch to the PostgreSQL shell
`psql`
-  Set a password for the newly created user for SonarQube database
`ALTER USER sonar WITH ENCRYPTED password 'sonar';`
-  Create a new database for PostgreSQL database by running:
`CREATE DATABASE sonarqube OWNER sonar;`
-  Grant all privileges to sonar user on sonarqube Database.
`grant all privileges on DATABASE sonarqube to sonar;`
-  Exit from the psql shell:
`\q`
-  Switch back to the sudo user by running the exit command.
`exit`
### Install SonarQube on Ubuntu 20.04 LTS
-  Navigate to the tmp directory to temporarily download the installation files
`cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip`
-  Unzip the archive setup to /opt directory
`sudo unzip sonarqube-7.9.3.zip -d /opt`
-  Move extracted setup to /opt/sonarqube directory
`sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube`


# CONFIGURE SONARQUBE
>  We cannot run SonarQube as a root user, if you run using root user it will stop automatically. The ideal approach will be to create a separate group and a user to run SonarQube

-  Create a group sonar
`sudo groupadd sonar`
-  Now add a user with control over the /opt/sonarqube directory
```
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```
-  Open SonarQube configuration file using your favourite text editor (e.g., nano or vim)
`sudo vim /opt/sonarqube/conf/sonar.properties`
-  Find the following lines:
```
#sonar.jdbc.username=
#sonar.jdbc.password=
```
Uncomment them and provide the values of PostgreSQL Database username and password:
```
#--------------------------------------------------------------------------------------------------

# DATABASE

#

# IMPORTANT:

# - The embedded H2 database is used by default. It is recommended for tests but not for

#   production use. Supported databases are Oracle, PostgreSQL and Microsoft SQLServer.

# - Changes to database connection URL (sonar.jdbc.url) can affect SonarSource licensed products.

# User credentials.

# Permissions to create tables, indices and triggers must be granted to JDBC user.

# The schema must be created first.

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
-  Edit the sonar script file and set RUN_AS_USER
`sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh`
```
# If specified, the Wrapper will be run as the specified user.

# IMPORTANT - Make sure that the user has the required privileges to write

#  the PID file and wrapper.log files.  Failure to be able to write the log

#  file will cause the Wrapper to exit without any way to write out an error

#  message.

# NOTE - This will set the user which is used to run the Wrapper as well as

#  the JVM and is not useful in situations where a privileged resource or

#  port needs to be allocated prior to the user being changed.

RUN_AS_USER=sonar
```
-  Now, to start SonarQube we need to do following:
    -  Switch to sonar user: `sudo su sonar`
    -  Move to the script directory: `cd /opt/sonarqube/bin/linux-x86-64/`
    -  Run the script to start SonarQube: `./sonar.sh start`
    -  Expected output shall be as:
```
Starting SonarQube...

Started SonarQube
```
Check SonarQube running status: `./sonar.sh status`
Sample Output below:
```
./sonar.sh status

SonarQube is running (176483).
```
-  To check SonarQube logs, navigate to /opt/sonarqube/logs/sonar.log directory
`tail /opt/sonarqube/logs/sonar.log`

Output
```
INFO  app[][o.s.a.ProcessLauncherImpl] Launch process[[key='ce', ipcIndex=3, logFilenamePrefix=ce]] from [/opt/sonarqube]: /usr/lib/jvm/java-11-openjdk-amd64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/opt/sonarqube/temp --add-opens=java.base/java.util=ALL-UNNAMED -Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Dhttp.nonProxyHosts=localhost|127.*|[::1] -cp ./lib/common/*:/opt/sonarqube/lib/jdbc/h2/h2-1.3.176.jar org.sonar.ce.app.CeServer /opt/sonarqube/temp/sq-process15059956114837198848properties

 INFO  app[][o.s.a.SchedulerImpl] Process[ce] is up

 INFO  app[][o.s.a.SchedulerImpl] SonarQube is up
```
You can see that SonarQube is up and running

-  Configure SonarQube to run as a systemd service
-  Stop the currently running SonarQube service
`cd /opt/sonarqube/bin/linux-x86-64/`
Run the script to start SonarQube
`./sonar.sh stop`
-  Create a systemd service file for SonarQube to run as System Startup.
`sudo nano /etc/systemd/system/sonar.service`
-  Add the configuration below for systemd to determine how to start, stop, check status, or restart the SonarQube service.
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```
-  Save the file and control the service with systemctl
```
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
```
### Access SonarQube
To access SonarQube using browser, type server’s IP address followed by port 9000.
http://server_IP:9000 OR http://localhost:9000.
Login to SonarQube with default administrator username and password – admin.
Now, when SonarQube is up and running, it is time to setup our Quality gate in Jenkins.

#  CONFIGURE SONARQUBE AND JENKINS FOR QUALITY GATE
-  In Jenkins, install SonarScanner plugin
-  Navigate to configure system in Jenkins. Add SonarQube server as shown below: `Manage Jenkins > Configure System`

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f956b7bc-4de0-41cb-8e40-3ae7998d44c1)
-----
-  Generate authentication token in SonarQube: `User > My Account > Security > Generate Tokens`
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/70a3f87e-1ab1-46a3-b812-eb1106f4a1b4)
-----
-  Configure Quality Gate Jenkins Webhook in SonarQube – The URL should point to your Jenkins server http://{JENKINS_HOST}/sonarqube-webhook/: `Administration > Configuration > Webhooks > Create`
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/14ed70ca-803d-4790-b243-431267685fcf)
-----
-  Setup SonarQube scanner from Jenkins – Global Tool Configuration
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/071b29c6-40ab-4d32-986a-ff7f4427065d)
-----

### Update Jenkins Pipeline to include SonarQube scanning and Quality Gate
-  Below is the snippet for a Quality Gate stage in Jenkinsfile.
```
stage('SonarQube Quality Gate') {
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }

        }
    }
```
>  NOTE: The above step will fail because we have not updated `sonar-scanner.properties
-  Configure sonar-scanner.properties – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the tools directory on the server to configure the properties file in which SonarQube will require to function during pipeline execution: `cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/`
- Open sonar-scanner.properties file: `sudo vi sonar-scanner.properties`
- Add configuration related to php-todo project
```
sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
```
>  HINT: To know what exactly to put inside the sonar-scanner.properties file, SonarQube has a configurations page where you can get some directions.
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f64f1c04-150f-4bfc-9ca6-c620f0f7301b)
-----
>  A brief explanation of what is going on the the stage – set the environment variable for the scannerHome use the same name used when you configured SonarQube Scanner from Jenkins Global Tool Configuration. If you remember, the name was SonarQubeScanner. Then, within the steps use shell to run the scanner from bin directory.

-  To further examine the configuration of the scanner tool on the Jenkins server – navigate into the tools directory
`cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/bin`
-  List the content to see the scanner tool sonar-scanner. That is what we are calling in the pipeline script.
-  Output of  `ls -latr`
```
ubuntu@ip-172-31-16-176:/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/bin$ ls -latr
total 24
-rwxr-xr-x 1 jenkins jenkins 2550 Oct  2 12:42 sonar-scanner.bat
-rwxr-xr-x 1 jenkins jenkins  586 Oct  2 12:42 sonar-scanner-debug.bat
-rwxr-xr-x 1 jenkins jenkins  662 Oct  2 12:42 sonar-scanner-debug
-rwxr-xr-x 1 jenkins jenkins 1823 Oct  2 12:42 sonar-scanner
drwxr-xr-x 2 jenkins jenkins 4096 Dec 26 18:42 .
```
-  Let's generate the jenkins configurations ourselves
-  To generate Jenkins code, navigate to the dashboard for the php-todo pipeline and click on the Pipeline Syntax menu item
`Dashboard > php-todo > Pipeline Syntax`
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/0dfabc06-8c0d-491a-9bd8-3808ee4d92bf)
-----
-  Click on Steps and select withSonarQubeEnv – This appears in the list because of the previous SonarQube configurations you have done in Jenkins. Otherwise, it would not be there.

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/af3bc41a-6e9d-4922-bb1c-d1168fd9ad1a)
------
-  Within the generated block, you will use the sh command to run shell on the server. https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/


### End-to-End Pipeline Overview
-  Sonarqube
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ea9c9cd0-353a-45ad-896a-c122ca68f3f7)
-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5f8703de-3454-405c-ae75-89e7152fa3b3)
-----

-  Jenkins
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/aac09811-50e9-4aab-b8b1-80170f063f97)


Our code has bugs as we can see, so we shouldn't push to production yet. We need to add a condition to enroce that it passes quality gate test before it deploys both the artifact and to an environment.

-  Let us update our Jenkinsfile to implement this:
-  First, we will include a When condition to run Quality Gate whenever the running branch is either develop, hotfix, release, main, or master: `when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}`
-  Then we add a timeout step to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable.
```
    timeout(time: 1, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
    }
```
-  The complete stage will now look like this:
```
    stage('SonarQube Quality Gate') {
      when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
            }
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
```
-  To test, create different branches and push to GitHub. You will realise that only branches other than develop, hotfix, release, main, or master will be able to deploy the code.

-  If the branch name is not among those specified above, it should look like this:
------
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8743b590-23ee-473e-8bd5-7ada4dcf91bc)
------
-  If the branch is among those listed, it will first pass the quality gate test before proceeding, if it passes, it deploys to environment and deploy artifact and if not, it won't run the rest of the codes. See both pictures

-----
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d5d6a8eb-234d-43d6-b3bf-a0a645d0182a)
------
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2ff85c84-8bf7-45e7-969b-aba5aa67d213)
------

>  Notice that with the current state of the code, it cannot be deployed to Integration environments due to its quality. In the real world, DevOps engineers will push this back to developers to work on the code further, based on SonarQube quality report. Once everything is good with code quality, the pipeline will pass and proceed with sipping the codes further to a higher environment.

-  Artifactory
------
![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/34b3c19e-1206-4fa2-9599-45708f6ec1ba)
------
