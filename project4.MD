# MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

- Set up your AWS Environment. It is recommended to use UBUNTU 20.
- ```sudo apt update```
- ```sudo apt upgrade```
- Add certificates: 
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
- Install apt install -y nodejs
![image](https://user-images.githubusercontent.com/29708657/223324269-e5824f49-c51d-49a0-8064-644b9dc785bf.png)

> Install mongodb
- ```sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6```
- ```echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list```
- install mongodb: ```sudo apt install -y mongodb```
- start the server: ```sudo service mongodb start```
- verify that the service is up and running: ```sudo systemctl status mongodb```
- install npm: ```sudo apt install -y npm```
- install body parser: ```sudo npm install body-parser```
- create a folder named Books: ```mkdir Book && cd Books```
- ```npm init```
- add a file to it named server.js: ```vi server.js```
- copy and paste the following into it:
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
