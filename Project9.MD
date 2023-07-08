# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

### Step 1 – Install Jenkins server
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”
- Install JDK (since Jenkins is a Java-based application):
```
sudo apt update
sudo apt install default-jdk-headless
```
- Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f4f0f39c-9310-4a26-96f1-8c3362f241fc">

- By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
- Perform initial Jenkins setup. From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080. You will be prompted to provide a default admin password

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/99852fee-6314-4471-ae64-71e0517a0252">

- Retrieve it from your server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
- Then you will be asked which plugins to install – choose suggested plugins.
- Once plugins installation is done – create an admin user and you will get your Jenkins server address.

- <img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/1c084ef5-ce43-4bef-9b04-e955fdb6d587">

### Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
- Enable webhooks in your GitHub repository settings

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/4033e5c5-aafc-4243-8bda-4b31958d7214">

- Go to Jenkins web console, click “New Item” and create a “Freestyle project”
- To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself
- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
- Save the configuration and let us try to run the build. For now we can only do it manually.
Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under `#1`

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5f3fb80d-d0b1-49c6-99db-335efc61e91d">

- Click “Configure” your job/project and add these two configurations
  - Configure triggering the job from GitHub webhook:

   ![image](https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/3dde4ee1-31df-448b-8483-f69788d0723b)

 - Configure “Post-build Actions” to archive all the files – files resulted from a build are called “artifacts”.

> Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/0e17c7d2-c23e-4971-8204-30b966101881">

- By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`
