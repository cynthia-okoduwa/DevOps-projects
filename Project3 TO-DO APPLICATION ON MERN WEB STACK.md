## SIMPLE WEB STACK IMPLEMENTATION (LEMP STACK)
### Task - To deploy a simple To-Do application that creates To-Do lists
### STEP 1 – BACKEND CONFIGURATION
#### Steps
* Update ubuntu: `sudo apt update`
* Upgrade ubuntu: `sudo apt upgrade`
* Get the location of Node.js software from Ubuntu repositories. Run: `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`
![MERNpix1](https://user-images.githubusercontent.com/74002629/177166115-965f0559-1c56-4f94-978b-edecab9f6b0d.PNG)

* Install Node.js on the server with this command: `sudo apt-get install -y nodejs` This command installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts. 
* Verify the node installation with this command: `node -v`
* Verify the npm installation with this command: `npm -v`
![MERNpix2](https://user-images.githubusercontent.com/74002629/177169274-46cc3cf7-108e-4521-b61f-67743e9e8b50.PNG)

##### Application Code Setup
* Create a new directory for your To-Do project: `mkdir Todo`
* Run the command below to verify that the Todo directory is created, run: `ls`
* Next, change the current directory to the newly created one: `cd Todo`
* Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Run the command: `npm init` then follow the prompt and finally answer yes to write out the package file.
![MERNpix3](https://user-images.githubusercontent.com/74002629/177170233-e1e5842c-d005-42e0-a98d-9024ccf9fcb1.PNG)
* Run the command `ls` to confirm that you have package.json file created.
![MERNpix4](https://user-images.githubusercontent.com/74002629/177170737-eb8054fc-1430-466c-a08f-6cfe408d3630.PNG)

##### INSTALL EXPRESSJS
* To use express, install it using npm: `npm install express`
* Next, create a file index.js with this command: `touch index.js`
* Run `ls` to confirm that your **index.js** file is successfully created.
![MERNpix5](https://user-images.githubusercontent.com/74002629/177171511-19fcd200-51d0-4bfc-a2ca-d04f829492b8.PNG)

* Next step is to install the **dotenv** module. Run this code: `npm install dotenv`
* Then open the **index.js** file with this command: `vim index.js`
* Type the code below into it and save:
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
* Use **:w** to save in vim and use **:qa** to exit vim
![MERNpix6](https://user-images.githubusercontent.com/74002629/177172327-27a3abb4-ee04-41bd-90dd-e3bf0a480cc9.PNG)

* Test start the server to see if it works. Type: `node index.js`
* You should see Server running on port 5000 in the terminal.
![MERNpix7](https://user-images.githubusercontent.com/74002629/177172683-a6e0794f-e599-4e30-8ebf-159e3c2fddf0.PNG)

* Next, open port 5000 in EC2 Security Groups and save changes.
* Open a browser and access the server’s Public IP or Public DNS name followed by port 5000: `http://<PublicIP-or-PublicDNS>:5000`
![MERNpix8](https://user-images.githubusercontent.com/74002629/177174585-ee179ca7-bda4-4ef4-a06e-f99a3d2529b1.PNG)

##### Routes
* The To-Do application needs to be able to complete 3 actions:
  * Create a new task
  * Display list of all tasks
  * Delete a completed task
* For each task, we need to create routes that will define various endpoints that the To-do app will depend on. Create a folder called **routes** with this command `mkdir routes`
* Change directory to **routes** folder and create a file **api.js** with the command: `touch api.js`
![MERNpix9](https://user-images.githubusercontent.com/74002629/177192281-e43015e1-def9-45fd-9a44-f218a80f85a5.PNG)

* Open the file with the command below `vim api.js`
* Copy and save the code below into the file:
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
![Project3pix7](https://user-images.githubusercontent.com/74002629/178157087-65c46d72-f37c-4abb-b54e-0928c5859093.PNG)

##### MODELS
* To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier. Change directory back Todo folder with `cd ..` and install Mongoose with the following command: `npm install mongoose`
* Create a new folder **models**, then change directory into the newly created **models** folder, Inside the models folder, create a file and name it **todo.js** with the following command: `mkdir models && cd models && touch todo.js`
* Open the file created with vim todo.js then paste the code below in the file:
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
![Project3pix8](https://user-images.githubusercontent.com/74002629/178157090-b669e929-07e1-4735-b13a-7a26ee500df0.PNG)

* Next, we update our routes from the file api.js in ‘routes’ directory to make use of the new model. In routes directory, open api.js with **vim api.js**, delete the code inside with `:%d` command and paste there code below into it then save and exit
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
![Project3pix9](https://user-images.githubusercontent.com/74002629/178157101-3211a910-daac-431e-96c7-4265ddbe034f.PNG)

##### MONGODB DATABASE
* A database is required where data will be stored. For this we will make use of mLab. Sign up for a shared clusters free account, Sign up on https://www.mongodb.com/atlas-signup-from-mlab. Follow the sign up process, select AWS as the cloud provider, and choose a region.
* For the purposes of this project, allow access to the MongoDB database from anywhere.
* Make sure you change the time of deleting the entry from 6 Hours to 1 Week
* Create a MongoDB database and collection inside mLab
![Project3pix10](https://user-images.githubusercontent.com/74002629/178157103-61393f4f-89da-4382-bdf1-425d50e5ae64.PNG)

* Next, in the index.js file, we specified **process.env** to access environment variables, but we are yet to create the file. Now, create a file in the **Todo** directory and name it **.env** To do this type: 
```
touch .env
vi .env
```
* Then add the connection string to access the database in it, just as below: 
`DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'`
* Update the **index.js** to reflect the use of **.env** so that Node.js can connect to the database. Delete existing content in the file, and update it with the following steps:
* using vim, follow below steps: Open the file with `vim index.js` and enter. Type`:` then type `%d` and enter. this will delete the entire content. Next, press `i` to enter the insert mode in vim. then, paste the entire code below in the file:
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
* Next, start the server using the command: `node index.js`
![Project3pix12](https://user-images.githubusercontent.com/74002629/178157109-37edc16f-a217-4d5c-8fe7-a26d79d5d708.PNG)

* You shall see a message **Database connected successfully**, if so – we have our backend configured. Now we are going to test it.

##### Testing Backend Code without Frontend using RESTful API
* Beacause we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code. We will use **Postman** to test our API.
* Create a **POST** request to the API `http://<PublicIP-or-PublicDNS>:5000/api/todos` This request sends a new task to the To-Do list so the application could store it in the database. Also set header key **Content-Type** as **application/json**
![Project3pix13](https://user-images.githubusercontent.com/74002629/178157113-44450534-533d-4f0a-b3f4-28a47eabe997.PNG)

* Create a GET request to your API on `http://<PublicIP-or-PublicDNS>:5000/api/todos` This request retrieves all existing records from the To-do application. 
![Project3pix14](https://user-images.githubusercontent.com/74002629/178157115-d56496b0-963c-4e61-a225-b7a2dcbd43a0.PNG)

### Step 2 Frontend Creation
* Having completed the functionality of the backend, it is time to create a user interface for a Web client (browser) to interact with the application via API.
* In the same root directory as your backend code, which is the Todo directory, run: `npx create-react-app client` to create a new folder in your Todo directory called **client**
##### Running a React App
* Before testing the react app, there are some dependencies that need to be installed: 
  * Install concurrently, used to run more than one command simultaneously from the same terminal window. `npm install concurrently --save-dev`
  ![Project3pix15](https://user-images.githubusercontent.com/74002629/178157404-46662611-72b7-4815-bba6-027735a0f529.PNG)
  
  * Install nodemon, used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes. `npm install nodemon --save-dev`
  * In Todo folder open the **package.json** file. make changes to the **script** replace with the code below:
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
##### Configure Proxy in package.json
* Change directory to **client**: `cd client`
* Open the package.json file: `vi package.json`
* Add the key value pair in the package.json file `"proxy": "http://localhost:5000"` The purpose of this is to ensure access to the application directly from the browser by simply calling the server url like **http://localhost:5000** rather than always including the entire path like **http://localhost:5000/api/todos**
![Project3pix16](https://user-images.githubusercontent.com/74002629/178157406-6c92c69c-17ab-46b5-ba89-c3f1a8b8f58c.PNG)

* Navigate to the Todo directory and run: **npm run dev** The app should open and start running on localhost:3000
![Project3pix17](https://user-images.githubusercontent.com/74002629/178157411-9c42cf2a-e341-4354-adb4-877143615238.PNG)
![Project3pix18](https://user-images.githubusercontent.com/74002629/178157412-47c64568-4685-454c-a487-0f579a5118d9.PNG)

* To access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule.

##### Creating your React Components
* From your Todo directory run: `cd client`
* move to the src directory: `cd src`
* Inside your src folder create another folder called components: `mkdir components`
* Move into the components directory with: `cd components`
* Inside ‘components’ directory create three files **Input.js**, **ListTodo.js** and **Todo.js**: `touch Input.js ListTodo.js Todo.js`
* Open Input.js file: `vi Input.js`
* Paste the following: 
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
![Project3pix19](https://user-images.githubusercontent.com/74002629/178157415-98401d13-c511-4fa0-96e9-0efb7fd3ce20.PNG)

* Move back to the client folder : `cd ../..`
* In the client folder, install Axios: `npm install axios`
![Project3pix20](https://user-images.githubusercontent.com/74002629/178157419-58d5c4d8-adf0-42a2-99bb-08e01c4aee35.PNG)

* Next, go to components directory: `cd src/components`
* Then open your **ListTodo.js**: `vi ListTodo.js`
* Paste the following code into the ListTodo.js file:
```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
```
* In in the Todo.js file you write the following code:
```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```
* Delete the logo and adjust our App.js. Navigate back to the **src** directory
* In the src folder run: `vi App.js`
* Copy and paste the code below into it
```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```
* Next, in the src directory open the App.css using: `vi App.css`
* Then paste the following code into App.css:
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```
* Next, in the src directory open the index.css: `vim index.css`
* Copy and paste the code below:
```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```
* Go to the Todo directory: `cd ../..`
* When you are in the Todo directory run: `npm run dev`
![Project3pix23](https://user-images.githubusercontent.com/74002629/178183189-a96c8436-e168-4625-a06e-ed2a9c229150.PNG)

* Assuming no errors when saving all these files, our To-Do app should be ready and fully functional with all the functionality working: creating a task, deleting a task and viewing all your tasks.
![Project3pix24](https://user-images.githubusercontent.com/74002629/178183198-07a6fa06-a735-4473-9df3-5ec427474a0e.PNG)





