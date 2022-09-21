## MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

I created an EC2 instance of t2.micro with Ubuntu server 20.4 LTS (HVM) image in AWS management console

## Step 1: Install NodeJs

Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. This will be used to set up the Express routes and AngularJS controllers.

1. I updated and upgraded Ubuntun by running the following commands
2. To Update 
`sudo apt update`
3. To Upgrade
`sudo apt upgrade`

4. Then I added certificates before installing NodeJs
`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

5. Install NodeJS
`sudo apt install -y nodejs`

## Step 2: Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time.

1. I ran the following command to to receive key from Ubuntu and add that to trusted set of keys
`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

2. Install MongoDB
`sudo apt install -y mongodb`
3. Start The server
`sudo service mongodb start`
4. Verify that the service is up and running
`sudo systemctl status mongodb`
5. Install npm – Node package manager.
`sudo apt install -y npm`
6. Install body-parser package.
We need ‘body-parser’ package to help us process JSON files passed in requests to the server.
`sudo npm install body-parser`
7. Create a folder named ‘Books’ and change the directory to Books 
`mkdir Books && cd Books`
8. In the Books directory, Initialize npm project
`npm init`
9. Add a file to it named server.js
`vim server.js`
10. Copy and paste the web server code below into the server.js file.
`var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});`

## Step 3: Install Express and set up routes to the server
1. I will use Express in to pass book information to and from MongoDB database. I will also use Mongoose package which provides a straight-forward, schema-based solution to model application data. I will use Mongoose to establish a schema for the database to store data of book register.
`sudo npm install express mongoose`
2. In ‘Books’ folder, created a folder named apps and changed the directory to apps
3. I created a file named routes.js
`touch routes.js`
`vim routes.js`
4. Copy and paste the code below into routes.js
`var Book = require('./models/book');
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
};`
5. In the ‘apps’ folder, I created a folder named models and changed the directory to models
`mkdir models && cd models`
6. Create a file named book.js
`touch book.js`
`vim book.js`
7. Copy and paste the code below into ‘book.js’
`var mongoose = require('mongoose');
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
module.exports = mongoose.model('Book', bookSchema);`

## Step 4 – Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in our web applications. I will use AngularJS to connect web page with Express and perform actions on our book register.

1. I changed the directory back to 'Books' by running the code
`cd ../..`
2. I created a folder named public and changed the directory to public
3. I added a file named script.js and edited it with vim text editor
`touch script.js`
`vim script.js`
4. Copy and paste the Code below (controller configuration defined) into the script.js file.
`var app = angular.module('myApp', []);
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
});`
5. In public folder, create a file named index.html;
`touch index.html`
`vim index.html`
6. Copy and paste the code below into index.html file.
`<!doctype html>
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
</html>`

5. Change the directory back up to Books
`cd ..`
6. Start the server by running this command:
`node server.js`

7. As the server is running, I launched a seperate SSH console to test what curl command returns locally.
`curl -s http://localhost:3300`
8. I need to go to EC2 instance and edit inbound rule in security group and add a new custom tcp with port 3300
9. To find my public ip address, I can get it from EC2 instance in AWS management console or I can run the following code to get it
`curl -s http://169.254.169.254/latest/meta-data/public-ipv4`

You can see the screenshot of the output in the Images folder.

**Thank you**










