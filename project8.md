
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

Ensure that Apache2 is up and running:

![Screen Shot 2021-05-30 at 3 44 45 PM](https://user-images.githubusercontent.com/44268796/120117968-0632a100-c15e-11eb-89ad-b2eadbdf13f7.png)


#### Step 3: Configure Load Balancer:

```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server

sudo systemctl restart apache2
```

##### Verify the configuration to see if the Load Balancer is serving as the entry point for allowing traffic into and out of the two web servers. Type the Public IP address or DNS name of the load balancer in the browser as follows:

```
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
```

The tooling website is now accessible through the Load Balancer



![Screen Shot 2021-05-30 at 3 52 52 PM](https://user-images.githubusercontent.com/44268796/120118292-be147e00-c15f-11eb-9320-c33125da3e58.png)




![Screen Shot 2021-05-30 at 3 53 33 PM](https://user-images.githubusercontent.com/44268796/120118294-bf45ab00-c15f-11eb-9960-65e9c3d4b328.png)


For project 7, I mounted /var/log/httpd/ from the Web Servers to the NFS server. I unmounted them and created a separate directory for each web server.
Then, I accessed the logs on each web server:

```
sudo tail -f /var/log/httpd/access_log
```
On Web Server 1:

![Screen Shot 2021-05-30 at 4 04 45 PM](https://user-images.githubusercontent.com/44268796/120118541-17c97800-c161-11eb-8624-fecbb91cb290.png)

On Web Server 2:

![Screen Shot 2021-05-30 at 4 04 34 PM](https://user-images.githubusercontent.com/44268796/120118539-16984b00-c161-11eb-93b3-705bc020a91b.png)



I also noticed how the logs kept adding new entries every time I refreshed the http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php on the web browser. The traffic was distributed between the two web servers. 
 
##### Local DNS Names Resolution:
 
 It can get tedious to maintain a long list of IP addresses for the web servers. We can configure local domain name resolution by adding entries to the /etc/hosts file. We can configure IP address to domain name mapping for the Load Balancer.
 
 ```
 sudo vi /etc/hosts
 ```
 Add the following entries:
 ```
 <WebServer1-Private-IP-Address> Web1
 <WebServer2-Private-IP-Address> Web2
 ```
  
  Update the configuration in the Load Balancer configuration file:
  ```
  sudo nano /etc/apache2/sites-available/000-default.conf
  ```
  Modify the IP address with the local domain names as follows:
  ```
  BalancerMember http://Web1:80 loadfactor=5 timeout=1
  BalancerMember http://Web2:80 loadfactor=5 timeout=1
  ```
  
  ![Screen Shot 2021-05-30 at 4 24 30 PM](https://user-images.githubusercontent.com/44268796/120119042-860f3a00-c163-11eb-83a3-b07b34a7b723.png)

  
  Upon using the curl command, the web server is accessible on the load balancer:
  
  ```
  curl http://Web1
  ```
  
  ![Screen Shot 2021-05-30 at 4 28 57 PM](https://user-images.githubusercontent.com/44268796/120119159-24030480-c164-11eb-8ca4-13884986b87e.png)
  
  This is internal configuration and thus local to the LB server, they will not be resolvable from other servers internally nor from the internet.
  
  
  Credits: https://darey.io
  
  
 
 


 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 





