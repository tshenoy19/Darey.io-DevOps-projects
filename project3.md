
# Simple To-Do application on MERN Web Stack

## Step 0 - Preparing prerequisites
 On AWS, create a new EC2 Instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image
 
## Step 1: Backend Configuration

Update Ubuntu and Upgrade Ubuntu
```bash
sudo apt update
sudo apt upgrade

```

Get location of Node.js software from Ubuntu repositories
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Install Node.js and npm with the following command:
```
sudo apt-get install -y nodejs
```
### Application Code Setup

Create a directory for the To-Do Project:
```
mkdir Todo
```
Change Directory:
```
cd Todo
```
To create a package.json file, run the command:
```
npm init
```
### Install ExpressJS, the serverside framework for Node.js
```
npm install express
```
Create a file called index.js
```
touch index.js
```
Install the dotenv module:
```
npm install dotenv
```

Open the vim editor and add the following code to index.js:
```js
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
The server should be running on port 5000 on the terminal


![Screen Shot 2021-04-22 at 7 35 07 AM](https://user-images.githubusercontent.com/44268796/115707911-5471aa80-a33d-11eb-8c65-38590ab80af0.png)

Update the Security Group on the EC2 instance to all Custom TCP traffic on Port 5000:


![Screen Shot 2021-04-22 at 7 38 38 AM](https://user-images.githubusercontent.com/44268796/115708401-ef6a8480-a33d-11eb-8456-4d58b35de1ce.png)

Open the browser and access Port 5000 on the server's public IP address or DNS name:
```
http://<PublicIP-or-PublicDNS>:5000
```

![Screen Shot 2021-04-22 at 7 42 38 AM](https://user-images.githubusercontent.com/44268796/115708707-5425df00-a33e-11eb-8255-2794d1d7ef2e.png)

### Creating Routes

There are three actions that the To-Do Application needs to be able to accomplish:
1. Create a new task
2. Display list of all tasks
3. Delete a completed task

For each task, we will define different endpoints using standard HTTP request methods: GET, PUT and DELETE
For each task, we need to create routes that will define the endpoints.

Create a directory called 'routes'
```
mkdir routes
```
Change directory:
```
cd routes
```
Create a file called 'api.js' and use the vim editor to add the following code:
```js
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
### Models

MongoDB is used as the database. To create the model and schema of the database, install Mongoose, a Node.js package.

Go back to Todo directory to install Mongoose
```
npm install mongoose
```
Create directory called 'models', change directory to 'models' and create file called 'touch.js'
```
mkdir models && cd models && touch todo.js
```
With vim editor, add the following code to todo.js:
```js
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

Next step is to update the api.js file to use the new model. Delete the existing code in api.js and update with the code below:
```js
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

## Creating MongoDB Database
mLab is used as a DBaaS to create the MongoDB database. 


![Screen Shot 2021-04-22 at 11 22 42 AM](https://user-images.githubusercontent.com/44268796/115740707-17b5ab80-a35d-11eb-85a4-9ef667d9f90b.png)

Create a file called .env in the Todo directory and add a connection string to access the database inside it:
```
DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'
```

Delete the contents of index.js file and add the following code. This facilitates the connection between Node.js and the database.
```js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true })
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
Check if the database is connected:
```
node index.js
```


![Screen Shot 2021-04-22 at 5 23 37 PM](https://user-images.githubusercontent.com/44268796/115787178-c83ca300-a38f-11eb-97b0-1e9fa27a84b4.png)



## Testing Backend Code without Frontend using RESTful API

The backend code can now be tested using RESTful API. Install Postman to test the API. 

Create a POST request to the API at http://<PublicIP-or-PublicDNS>:5000/api/todos
 
 
![Screen Shot 2021-04-23 at 7 19 41 AM](https://user-images.githubusercontent.com/44268796/115864504-ed1f2d80-a404-11eb-937b-8f632712fb6a.png)
 


![Screen Shot 2021-04-23 at 7 20 36 AM](https://user-images.githubusercontent.com/44268796/115864543-fc9e7680-a404-11eb-9eeb-5765adabdff8.png)

































