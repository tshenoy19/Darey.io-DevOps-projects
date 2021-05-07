
### Implementation of Client-Server Architecture using MySQL Database Management System

##### Step 0: Launch two Linux-based EC2 instances to serve as the Server and the Client
```
Server A name - `mysql_server`
Server B name - `mysql_client`
```

##### Step 1: Set up the DB Server - mysql-server

Connect to mysql-server

###### Update the server 
```
sudo apt update -y
```
###### Install MySQL server software
```
sudo apt install mysql-server
```
###### Enable MySQL
```
sudo systemctl enable mysql
```

##### Step 3: Set up the Client Server- mysql-client

