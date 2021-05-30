
#### Load Balancer Solution With Apache

This project builds upon project 7 further by adding a Load Balancer to distribute the load evenly between the Web Servers. The users will access the application by using a single public IP address/DNS name of the Load Balancer instead of using separate IP addresses/ DNS names of each Web Server. This ensures a single point of entry to the application through the Load Balancer and also helps to create a better user experience by distributing the traffic evenly between the Web Servers. 


#### Task: To configure an Apache Load Balancer using an Ubuntu EC2 machine to serve the Tooling Website.

#### Prerequisites:
1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server


![Screen Shot 2021-05-30 at 3 25 54 PM](https://user-images.githubusercontent.com/44268796/120117479-5d834200-c15b-11eb-8e62-83823a06e7b0.png)


#### Configuring an Apache Load Balancer

##### Step 1: Create an Ubuntu Server 20.04 EC2 instance and name it as "apache-lb" The setup on AWS should look like this: 





