
### Implementation of Client-Server Architecture using MySQL Database Management System

##### Step 0: Launch two Linux-based EC2 instances to serve as the Server and the Client
```
Server A name - `mysql-server
Server B name - `mysql-client
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
###### Run the MySQL security script
```
sudo mysql_secure_installation
```
###### Run the MySQL command
```
sudo mysql
```


![Screen Shot 2021-05-07 at 2 22 21 PM](https://user-images.githubusercontent.com/44268796/117492510-ade1f800-af3f-11eb-9291-3062436b08fb.png)










##### Step 3: Set up the Client Server- mysql-client

###### Update the Server
```
sudo apt update -y
```
###### Install MySQL Client Server software
```
sudo apt install mysql-client -y
```
###### Update the Security Group of the DB server to allow traffic from the client's private IP address at port 3306



![Screen Shot 2021-05-07 at 2 13 28 PM](https://user-images.githubusercontent.com/44268796/117491720-8dfe0480-af3e-11eb-9520-8f79661d938f.png)






















