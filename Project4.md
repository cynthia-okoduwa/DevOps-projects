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
![Project4pix1](https://user-images.githubusercontent.com/74002629/177740062-bfa83002-6ee8-431a-81d2-0b5517fd470d.PNG)

  * Next we install nodejs with this: `sudo apt install -y nodejs`
  
 ![Project4pix2](https://user-images.githubusercontent.com/74002629/177740077-2135389b-62ec-423f-9a16-95ca7a29ce73.PNG)
  
2. ##### Install MongoDB
  * First we add our MongoDB key server with: `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`
  * Add repository: `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`
  * Install MongoDB with the following comand: `sudo apt install -y mongodb`
  
  ![Project4pix3](https://user-images.githubusercontent.com/74002629/177740109-72186618-747d-433a-9bdc-e4392be0f973.PNG)
  
  * Verify Server is up and running: `sudo systemctl status mongodb`
  
   ![Project4pix5](https://user-images.githubusercontent.com/74002629/177740147-d4fcae4f-a5e7-4d1a-86e2-7d480b1efd4b.PNG)
  
  * Install npm – Node package manager: `sudo apt install -y npm`
  * Next we install body-parser package to help with processing JSON files passed in requests to the server. Use the following command: `sudo npm install body-parser`
  
  ![Project4pix7](https://user-images.githubusercontent.com/74002629/177740193-5d033df2-8d50-4c05-9b66-0c23eb6e4875.PNG)
  
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
![Project4pix9](https://user-images.githubusercontent.com/74002629/177740233-afab3647-3e60-4c6c-beeb-b767b1c5357f.PNG)

3. ##### Install Express and set up routes to the server
 * Express will be used to pass book information to and from our MongoDB database and Mongoose will be used to establish a schema for the database to store data of our book register. To begin installation, type: `sudo npm install express mongoose` and enter.
 
 ![Project4pix10](https://user-images.githubusercontent.com/74002629/177740255-3fefa006-6bf8-4763-ad2e-de35e961312c.PNG)
 
 * while in **Books** folder, create a directory named **apps** and navigate into it with: `mkdir apps && cd apps`
 * Inside **apps**, create a file named routes.js with: `vi routes.js`
 * Copy and paste the code below into routes.js
 ```
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
 ```
 ![Project4pix11](https://user-images.githubusercontent.com/74002629/177742866-25909219-e92c-4dba-9f51-39937aed721e.PNG)
 
 * Also create a folder named **models** in the **apps** folder, then navigate into it: `mkdir models && cd models`
 * Inside **models**, create a file named **book.js** with: `vi book.js`
 * Copy and paste the code below into ‘book.js’
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```
![Project4pix12](https://user-images.githubusercontent.com/74002629/177742884-89cd7741-6b6c-4e07-be4f-ff36547032dd.PNG)

4. ##### Access the routes with AngularJS
 * The final step would be using AngularJS to connect our web page with Express and perform actions on our book register.
 * Navigate back to **Books** directory using: `cd ../..`
 * Now create a folder named **public** and move into it: `mkdir public && cd public`
 * Add a file named **script.js**: `vi script.js`
 * And copy and paste the following code:
 ```
 var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
![Project4pix13](https://user-images.githubusercontent.com/74002629/177742899-8172f3ca-bb34-4a30-815b-4bc76365d82b.PNG)

* Also in **public** folder, create a file named **index.html**: `vi index.html`
* And and paste the foloowing html code below into it: 
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
![Project4pix14](https://user-images.githubusercontent.com/74002629/177742928-12b6730f-d169-438c-90f9-bc4a8fe3724a.PNG)

* Then change the directory back up to **Books** using: `cd ..`
* Now, start the server by running this command: `node server.js` If all goes well server should be up and running and we can connect to it on port 3300.
* This however was not the case in my test, I kept on getting an error. see error in the picture below:
 
![Project4pix15](https://user-images.githubusercontent.com/74002629/177742997-92f09460-b0e5-4e8f-8f60-e79a6a1dd314.PNG)

* After several attempts at troubleshooting I figured out what the issue was. The issue was with my nodejs version. During the installation, I installed version 12 but for some reson version 12 won't work but keeps giving me errors when I attempt to start the sever.
* I solved this problem by upgrading my node version to version, 17.0.0
* After this I tried to start the server again and it ran successfully.

![Project4pix16](https://user-images.githubusercontent.com/74002629/177743041-af433cfa-94af-45ac-a05d-beecdc2b2fcc.PNG)

* Next I accessed the HTML page over the internet via port 3300 using the public IP: `http://34.200.223.160:3300/`

![Project4pix17](https://user-images.githubusercontent.com/74002629/177743063-c80a2fb6-0550-4319-b8ce-676277d0f58d.PNG)

* Finally I enter some data into the database and it reflected.

![Project4pix18](https://user-images.githubusercontent.com/74002629/177743087-9fcbe570-a55b-403c-9173-97feebdedaf9.PNG)



