
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

### Create a POST request to the API:
 
 
![Screen Shot 2021-04-23 at 7 19 41 AM](https://user-images.githubusercontent.com/44268796/115864504-ed1f2d80-a404-11eb-937b-8f632712fb6a.png)
 


![Screen Shot 2021-04-23 at 7 20 36 AM](https://user-images.githubusercontent.com/44268796/115864543-fc9e7680-a404-11eb-9eeb-5765adabdff8.png)


### Create a GET request on Postman:


![Screen Shot 2021-04-23 at 11 07 48 AM](https://user-images.githubusercontent.com/44268796/115891720-65491b80-a424-11eb-8099-d05a7d8fd542.png)

This confirms that the backend is working and responding to API calls.

## Step 2 - Frontend creation

To create the frontend application, install the create-react-app in the same root directory as the backend- Todo

```
npx create-react-app client
```

This creates a folder in the Todo directory called 'Client' which will host all the React code.

### Running the React App

Some dependencies need to be installed before testing the React app:

Install Concurrently:
```
npm install concurrently --save-dev
```

Install nodemon:
```
npm install nodemon --save-dev
```

In the package.json file within the Todo directory, replace the existing scripts with the following:


![Screen Shot 2021-04-23 at 11 20 31 AM](https://user-images.githubusercontent.com/44268796/115893333-0d131900-a426-11eb-9c5e-08089bd4cb62.png)



### Configure Proxy in package.json

Now to access the application from the browser by just using the URL, the proxy needs to be configured.

```
cd client
vi package.json
```

Add the following key-value pair:
```
"proxy": "http://localhost:5000"
```

Configure the security group on the EC2 instance to allow TCP on port 3000 from anywhere.

Change back to Todo directory:
```
cd Todo
npm run dev
```

The app should now be running on the local host at port 3000.


![Screen Shot 2021-04-23 at 11 39 44 AM](https://user-images.githubusercontent.com/44268796/115895610-9d525d80-a428-11eb-85db-041f16a07316.png)


### Creating React components

Next step is to create three components for the Todo app. Two of them will be stateful and one will be stateless. 

From the Todo directory:
```
cd client
cd src
mkdir components
cd components
touch Input.js ListTodo.js Todo.js
```

Use Vim to add the following code to Input.js file:
```js
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

Axios is a Promise based HTTP client for the browser and node.js. Move to the src folder and then the client folder and install it:
```
cd ..
cd ..
npm install axios
```

Next from the component directory, open the ListTodo.js file and add the following code:

```js
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

Add the following code to the Todo.js file:
```js
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

Next, we will change the appearance of the user interface by deleting the logo and making changes to the App.js file.

Go to src folder:
```
cd ..
```

Use Vim to add the following code to the App.js file:
```js
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

Open App.css in the src directory and add the following code:
```css
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

Open index.css in the src directory and add the following code:
```css
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

Now go to Todo directory and run the app:

```
npm run dev
```

The final To-Do app is ready and fully functional providing the user with the ability to create a task, delete a task and view tasks. 

































