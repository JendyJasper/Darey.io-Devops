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
