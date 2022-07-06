## SIMPLE TO-DO APPLICATION ON MERN WEB STACK
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

#### INSTALL EXPRESSJS
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

#### Routes
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

### MODELS
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
### MONGODB DATABASE
* A database is required where data will be stored. For this we will make use of mLab. Sign up for a shared clusters free account, Sign up on https://www.mongodb.com/atlas-signup-from-mlab. Follow the sign up process, select AWS as the cloud provider, and choose a region near you.
* For the purposes of this project, allow access to the MongoDB database from anywhere.
* Make sure you change the time of deleting the entry from 6 Hours to 1 Week
* Create a MongoDB database and collection inside mLab
