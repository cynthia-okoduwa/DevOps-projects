## SIMPLE TO-DO APPLICATION ON MERN WEB STACK
### STEP 1 – BACKEND CONFIGURATION
#### Steps
* Update ubuntu: **sudo apt update**
* Upgrade ubuntu: **sudo apt upgrade**
* Get the location of Node.js software from Ubuntu repositories. Run: **curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -**
* Install Node.js on the server with this command: **sudo apt-get install -y nodejs** This command installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.
* Verify the node installation with this command: **node -v**
* Verify the npm installation with this command: **npm -v**
##### Application Code Setup
* Create a new directory for your To-Do project: **mkdir Todo**
* Run the command below to verify that the Todo directory is created, run: **ls**
* Next, change the current directory to the newly created one: **cd Todo**
* Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Run the command: **npm init** then follow the prompt and final answer yes to write out the package file.
* Run the command **ls** to confirm that you have package.json file created.

