# MERN STACK IMPLEMENTATION

- update ubuntu: ```sudo apt update```
- upgrade ubuntu: ```sudo apt upgrade```
- Lets get the location of Node.js software from Ubuntu repositories: curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
- Install Node.js with the command: ```sudo apt-get install -y nodejs```
- run ```npm -v``` and ```node -v``` to confirm their installations
- run ```mkdir Todo``` to create a directory for the Todo project and ```cd Todo``` to switch to that directory
- run ```npm init``` to initiallise a nodejs project. This creates a package.json file. Keep following the prompts until you are done

### install Express JS
- install express: ```npm install express```
- create index.js: ```touch index.js```
- install the dotenv module: ```npm install dotenv```
- open the index.js file: ```vim index.js```
- type the below code into it and save:
```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
- we specified here to use port 5000
- save and exit vim ```:w``` and ```:qa```
- Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type: ```node index.js```
