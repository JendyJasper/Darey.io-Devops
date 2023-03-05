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
- go to aws security group and set it to allow connection from port 5000. 
- Access your public address, specifying the port 5000 and you wull see an Express page.


## Configure routes
Routes will handle different features we want the to do app to do, like to create, view and delete the todo. For each task, we need to create routes that will define various endpoints that the To-do app will depend on.

- mkdir routes
- cd routes
- create ```touch api.js```
- open the file ```vim api.js```
- copy and paste the below code:
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

- Change directory back Todo folder with cd .. and install Mongoose: ```npm install mongoose```
- Create a new folder models : ```mkdir models```
-Change directory into the newly created ‘models’ folder with: ```cd models```
Inside the model s folder, create a file and name it todo.js: ```touch todo.js```

> Tip: All three commands above can be defined in one line to be executed consequently with help of && operator, like this: ```mkdir models && cd models && touch todo.js```
Open the file created with vim todo.js then paste the code below in the file:
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
> Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model. In Routes directory, open api.js with ```vim api.js``` , delete the code inside with ```:%d``` command and paste there code below into it then save and exit: 
```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```

## Set up MONGO DB
- signup on https://www.mongodb.com/atlas-signup-from-mlab
- Create a new cluster or project, create a user, create a database, create a collection, and connect to your mongodb database to get your login credentials
> To kill a node js process from running in the background: ```sudo lsof -i :3000``` where 3000 is the port number. Get the process ID and run ```kill -9 <PID>```
- You can set up specifici IP addresses to access your database or set it up to allow access from anywhere
> In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now.
- Create a file in your Todo directory and name it .env. ```touch .env```
- Open the file ```nano .env```
- Add the connection string to access the database in it, just as below: ```DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'```. You can get this details by clicking the CONNECT button on the database page
- Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database. Simply delete existing content in the file, and update it with the entire code below.
> To do that using vim, follow below steps:
- Open the file with ```vim index.js```
- Press esc
- Type :
- Type %d
- Hit ‘Enter’
- The entire content will be deleted, then, Press i to enter the insert mode in vim
> Now, paste the entire code below in the file:
```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
- Start your server using the command: ```node index.js```
> You shall see a message ‘Database connected successfully’, if so – we have our backend configured. Now we are going to test it.
  
> Test using postman to send POST, GET and DELETE request. Use content-Type as application/json to send the post request. To use the DELETE request, pass in the ID of the item to the URI. link to use is http://35.171.193.174:5000/api/todos. You should be able to retriev and created todos and create new todos or even delete existing todos
  
## Frontend creation
  Let's create a user interface for our react app.

- we will use the create-react-app command to scaffold our app. In the same root directory as your backend code, which is the Todo directory, run: "npx create-react-app client". This will create a new folder in your Todo directory called client, where you will add all the react code.
- Before testing the react app, there are some dependencies that need to be installed.
- Install concurrently. It is used to run more than one command simultaneously from the same terminal window. ```npm install concurrently --save-dev```
- Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes. ```npm install nodemon --save-dev```
- In Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below.
 ```
  "scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
 ![image](https://user-images.githubusercontent.com/29708657/222961685-55352183-e710-42a5-8553-20647ebb1fab.png)

### Configure Proxy in package.json
- Change directory to ‘client’
- cd client
- Open the package.json file: ```vi package.json```
- Add the key value pair in the package.json file ```"proxy": "http://localhost:5000"```
> The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos
  
- Now, ensure you are inside the Todo directory, and simply do: ```npm run dev```

> Your app should open and start running on localhost:3000. Important note: In order to be able to access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule. 
  
### Creating your React Components
- Move to client directory: ```cd client```
- move to the src directory: ```cd src```
- Inside your src folder create another folder called components: ```mkdir components```
- Move into the components directory with: ```cd components```
- Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js.: ```touch Input.js ListTodo.js Todo.js```
- Open Input.js file: ```vi Input.js```
- copy and paste the following:
```
  import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```
  after that, do the following to use axion
- Move to the src folder ```cd ..```
- Move to clients folder: ```cd ..```
- Install Axios
- npm install axios
