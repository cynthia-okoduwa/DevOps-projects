## SIMPLE TO-DO APPLICATION ON MERN WEB STACK
### STEP 1 – BACKEND CONFIGURATION
#### Steps
* Update ubuntu: `sudo apt update`
* Upgrade ubuntu: `sudo apt upgrade`
* Get the location of Node.js software from Ubuntu repositories. Run: `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`
* Install Node.js on the server with this command: `sudo apt-get install -y nodejs` This command installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.
* Verify the node installation with this command: `node -v`
* Verify the npm installation with this command: `npm -v`
##### Application Code Setup
* Create a new directory for your To-Do project: `mkdir Todo`
* Run the command below to verify that the Todo directory is created, run: `ls`
* Next, change the current directory to the newly created one: `cd Todo`
* Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Run the command: `npm init` then follow the prompt and final answer yes to write out the package file.
* Run the command `ls` to confirm that you have package.json file created.

#### INSTALL EXPRESSJS
* To use express, install it using npm: `npm install express`
* Next, create a file index.js with this command: `touch index.js`
* Run `ls` to confirm that your **index.js** file is successfully created.
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
* Test start the server to see if it works. Type: `node index.js`
