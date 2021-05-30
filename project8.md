
#### Load Balancer Solution With Apache

This project builds upon project 7 further by adding a Load Balancer to distribute the load evenly between the Web Servers. The users will access the application by using a single public IP address/DNS name of the Load Balancer instead of using separate IP addresses/ DNS names of each Web Server. This ensures a single point of entry to the application through the Load Balancer and also helps to create a better user experience by distributing the traffic evenly between the Web Servers. 


#### Task: To configure an Apache Load Balancer using an Ubuntu EC2 machine to serve the Tooling Website.

#### Prerequisites:
1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server


![Screen Shot 2021-05-30 at 3 25 54 PM](https://user-images.githubusercontent.com/44268796/120117479-5d834200-c15b-11eb-8e62-83823a06e7b0.png)


#### Configuring an Apache Load Balancer

#### Step 1: Create an Ubuntu Server 20.04 EC2 instance and name it as "apache-lb" 
The setup on AWS should look like this: 

![Screen Shot 2021-05-30 at 3 33 20 PM](https://user-images.githubusercontent.com/44268796/120117691-5f013a00-c15c-11eb-9466-1ae366aa7707.png)


On the Apache Load Balancer EC2 instance, open TCP traffic inbound on Port 80. Also allow all traffic from load balancer to the web servers on their security group.

 #### Step 2: Install Apache 2 Load Balancer on the EC2 instance
 
 ```
 # Install Apache2
 sudo apt update
 sudo apt install apache2 -y
 sudo apt-get install libxml2-dev
 ```
 ```
 #Enable following modules:
 sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```











 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 





