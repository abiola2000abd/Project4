# Project 4 - MEAN STACK DEPLOYMENT TO UBUNTU 18.04TLS IN AWS
## Task
  Implement a simple Book Register web form using MEAN stack.

### Step 1: Install NodeJs

Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used  to set up the Express routes and AngularJS controllers.

Update ubuntu

`sudo apt update`

![update](./images/sudo-update.png)

Upgrade ubuntu

`sudo apt upgrade`

![upgrade](./images/sudo-upgrade1.png)

Type "yes" to continue

![upgrade2](./images/sudo-upgrade2.png)

### Add certificates

`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

![inst](./images/add-cert.png)

`curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

![nou](./images/cur1.png)


![sou2](./images/curl2.png)

 Next we Install NodeJS

`sudo apt install -y nodejs`

![nojs](./images/nojs.png)


![nojsd](./images/nojs2.png)


## Step 2: Installing MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
mages/WebConsole.gif

We add the MangoDB keyserver

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

![key](./images/sudo-apt-key.png)

Add the repository 

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

![echo](./images/echo.png)

Install MongoDB

`sudo apt install -y mongodb`

![inst-Mongo](./images/mongodb-ins.png)

After installing MongoDB we start The server to besure it properly install.

`sudo service mongodb start`


Verify that the service is up and running.

`sudo systemctl status mongodb`

![start](./images/mogo-started.png)

Install npm – Node package manager.

`sudo apt install -y npm`

![pakt](./images/apt-pack-manr.png)

Install body-parser package

We need ‘body-parser’ package to help us process JSON files passed in requests to the server.

`sudo npm install body-parser`

![body](./images/body-parser.png)

we need to Create a folder named ‘Books’

`mkdir Books && cd Books`

In the Books directory,we also Initialize npm project

`npm init`

![int](./images/creat.png)

Add a file to it named server.js

`touch server.js`

We then edit the server.js file.

`vi server.js`

![ser](./images/sever.png)

Copy and paste the web server code below into the server.js file.

```{
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

}
````
Save and exit

![cp](./images/serverjs.png)

### Step 3: Install Express and set up routes to the server

Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model our application data. We will use Mongoose to establish a schema for the database to store data of our book register.

`sudo npm install express mongoose`

![expre](./images/mangoo-inst.png)

We then create a folder named apps inside the BOOKs' Folder.

`mkdir apps && cd apps`

we also create a file name routes.js  inside the apps folder.After creating ,we need to open and edit the routes.js file.

`vi routes.js`

![routes](./images/cre-book.png)

copy and paste the code below into routes.js

```{
var Book = require('./models/book');
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

}
```
Save  and Exit

![copyied](./images/paste.png)

In the ‘apps’ folder, create a folder named models

`mkdir models && cd models`

Create a file named book.js inside the models folder. We also need to open and  edit the book.js file.

`vi book.js`


![bkjs](./images/book.png)


Copy and paste the code below into ‘book.js’ 


```
{
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
}
```
Save and Exit.

![editbook](./images/book-edit.png)

### Step 4 – Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in our web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

Change the directory back to ‘Books’

`cd ../..`

Create a folder named public 

`mkdir public && cd public`

Add a file named script.js , open and edit

`vi script.js`

![edit](./images/script%20-vi.png)

Copy and paste the Code below (controller configuration defined) into the script.js file.

```{
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
}
```
Save and exit after pasting.

![paste1](./images/index.js-edited.png)

Create a file named index.html inside the public folder,Open and edit the file.

`vi index.html`

![html](./images/html.png)

Cpoy and paste the code below into index.html file.

```{
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

}
````
![editp](./images/index-html-edit.png)

Change the directory back up to Books

`cd ..` 

Start the server by running this command:

`node server.js`

![ser-run](./images/server-runing.png)

The server is now up and running, we can connect it via port 3300. We can launch a separate Putty or SSH console to test what curl command returns locally.

curl -s http://localhost:3300

It shall return an HTML page, it is hardly readable in the CLI, but we can also try and access it from the Internet.


For this to work – We need to open TCP port 3300 in your AWS Web Console for your EC2 Instance.

![aws](./images/port3300.png)


We then use our Public IP Address

 http://18.170.88.69:3300

![uotput](./images/final%20output.png)
 
 Did some entries.

![input-data](./images/oupt-data.png)