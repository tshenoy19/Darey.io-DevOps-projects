
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


###### Create a remote user for the database
```
mysql> CREATE USER 'remote_user'@'%' IDENTIFIED BY 'password';
```
###### Create a database
```
mysql> CREATE DATABASE <dbname>;
```
###### Grant privileges
```
mysql> GRANT ALL ON <dbname>.* TO 'remote_user'@'%' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```
###### Configure MySQL server to allow connections from remote hosts

```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 
```
Change the bind address to allow traffic from 0.0.0.0


![Screen Shot 2021-05-07 at 2 42 19 PM](https://user-images.githubusercontent.com/44268796/117494572-80e31480-af42-11eb-8433-1f2a6b61f4a8.png)


###### Restart the MySQL service to activate the new configurations
```
sudo systemctl restart mysql
```

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


##### Step 4: Connect to the DB Server from the Remote Client Server

On client server:
```
mysql -u remote_user -h 172.31.42.198 -p 
```
Enter the password upon prompt. The connection is established and the Remote Client is able to access the DB Server.


![Screen Shot 2021-05-07 at 2 57 57 PM](https://user-images.githubusercontent.com/44268796/117496179-ab35d180-af44-11eb-9e4a-3e18183b9787.png)






















