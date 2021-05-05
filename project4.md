## Creating a MEAN STACK on AWS Cloud

### Objective: Create a simple Book Register web form using MEAN Stack

The MEAN stack uses:
MongoDB- a document database to store and retrieve data
Express- a back-end application framework used to make requests to the database
Angular- a front-end application framework that handles server and client requests
Node.js- a JavaScript run-time environment that accepts requests and displays results to the end user



#### Step 0: Preparing Prerequisites
Launch an EC2 instance with Ubuntu OS using Free Tier


#### Step 1: Install Node.js

Node.js is installed first to set up Express routes and AngularJS controllers

Connect to the Ubuntu server with SSH.

##### Update and upgrade Ubuntu

```
sudo apt update
sudo apt upgrade
```

##### Install Node.js
```
sudo apt install -y nodejs
```

#### Step 2: Install MongoDB

The MongoDB database will be used to contain book name, isbn number, author, and number of pages.

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
##### Install MongoDB
```
sudo apt install -y mongodb
```

##### Start and verify that the server is up and running
```
sudo service mongodb start
sudo systemctl status mongodb
```



![Screen Shot 2021-05-05 at 11 22 35 AM](https://user-images.githubusercontent.com/44268796/117166317-40de2f00-ad94-11eb-988a-5de9ac59ece1.png)


##### Install npm - Node package manager
```
sudo apt install -y npm
```
##### Install the body-parser package to help process JSON files passed in requests to the server
```
sudo npm install body-parser
```
##### Create a folder called 'Books'
```
mkdir Books && cd Books
```
##### Initialize npm project in 'Books' directory
```
npm init
```
##### Create a file called 'server.js' and add the following code:
```
vi server.js
```
```js
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

#### Step 3: Install Express and set up routes to the server

Express is used to pass book information to and from the database.
Mongoose package is used to create a schema for the database to store the data of the book register

##### Install Mongoose
```
sudo npm install express mongoose
```
##### Create a folder called 'apps' in the 'Books' directory
```
mkdir apps && cd apps
```
##### Create a file called 'routes.js' and add the following code
```
vi routes.js
```
```js
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
```



















