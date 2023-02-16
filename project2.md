# LEMP STACK IMPLEMENTATION


- Create an ec2 instance on AWS
- Run ```sudo apt update``` to update your packages
- run ```sudo apt install nginx``` to install nginx
- run ```sudo systemctl status nginx``` to confirm it's running as a service
- enable port 80 on your instance security group to allow web connections from anywhere
- test if the web server is running locally on your ubuntu seerver by typing ```curl http://localhost:80``` on your browser
- test if it can receive requests from the internet. Get your public ip address by running the command, ```curl -s http://169.254.169.254/latest/meta-data/public-ipv4```
- Install mysql database to manage data for the site by running ```sudo apt install mysql-server```
- login to mysql using ```sudo mysql```
- set a password for mysql root user ```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';```
- ```exit``` mysql
- run a security script that will remove insecure default settings ```sudo mysql_secure_installation```
- Don't enable ```VALIDATE PASSWORD``` if you don't want to set a specified criteria for passwords to be accepted
- 
- 
