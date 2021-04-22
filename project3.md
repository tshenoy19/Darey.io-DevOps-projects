
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

























