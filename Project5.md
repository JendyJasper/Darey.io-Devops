## IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)
- create two ec2 instances and configure them, name one "mysql server" and the other one "mysql client"
- log in to both instances, and run ```sudo apt update``` and ```sudo apt upgrade```
- on the mysql server ec2 instance, run ```sudo apt install mysql-server``` to install the mysql server software
- on the mysql client ec2 instance, run ```sudo apt install mysql-client``` to install the mysql client software
- go to your mysql server ec2 instance security group and edit the inbound rule to allow mysql connection from mysql client ec2 instance. Because they are both in the same local netowk, use the private address of the mysql client as the only allowed IP address to connect using mysql to the mysql server ec2 instance. Mysql uses port 3306
- <img width="1419" alt="image" src="https://user-images.githubusercontent.com/29708657/227425832-3c06d17f-97f8-4108-9e4b-f542f8470193.png">
