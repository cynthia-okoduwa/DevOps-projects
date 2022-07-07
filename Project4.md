## MEAN STACK DEPLOYMENT TO UBUNTU IN AWS
### Task- Implement a simple Book Register web form using MEAN stack.
#### Steps
1. ##### Install Nodejs
  * Provision Ubuntu 20.4 instance in AWS
  * Connect to the instance through an SSH client.
  * Once in the terminal, update Ubuntu using this command: `sudo apt update`
  * Next, upgrade Ubuntu with `sudo apt upgrade`
  * Add certificates: 
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
  * Next we install nodejs with this: `sudo apt install -y nodejs`
2. ##### Install MongoDB
  * First we add our MongoDB key server with: `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`
  * Add repository: `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`
  * Install MongoDB with the following comand: `sudo apt install -y mongodb`
  * Verify Server is up and running: `sudo systemctl status mongodb`
  * Install npm – Node package manager: `sudo apt install -y npm`
  * Next we install body-parser package to help with processing JSON files passed in requests to the server. Use the following command: `sudo npm install body-parser`
  * Next we create the **Books** directory and navigate into it with the following command: `mkdir Books && cd Books` 
  * Inside the Books directory initialize npm project and add a file to it with the following command: `npm init` Then add **sever.js** file with: `vi server.js`
  * In the server.js file, paste the following code:
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

