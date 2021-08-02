## AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology

The goal of this project is to build a secure infrastructure inside the AWS VPC network for a fictitious company that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) for the DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.
  
![Screen Shot 2021-07-06 at 10 02 53 AM](https://user-images.githubusercontent.com/44268796/124613547-57127500-de41-11eb-9ffa-b3888bcfcee2.png)

#### Starting Off the AWS Cloud Project
  
There are few requirements that must be met before starting the project:

1. Properly configure the AWS account and Organization Unit. Use this [video](https://www.youtube.com/watch?v=9PQYCc_20-Q) to implement the setup.
2. Create an AWS Master account. (Also known as Root Account)
3. Within the Root account, create a sub-account and name it DevOps. (A different email address is required to complete this)
4. Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (The Dev resources will be launched in there)
  
  ![Screen Shot 2021-07-06 at 10 32 00 AM](https://user-images.githubusercontent.com/44268796/124619159-67791e80-de46-11eb-859b-335259da025f.png)

5. Move the DevOps account into the Dev OU.
  
  ![Screen Shot 2021-07-06 at 10 40 15 AM](https://user-images.githubusercontent.com/44268796/124619324-8f688200-de46-11eb-8b90-a7768b45bd2f.png)


6. Login to the newly created AWS account using the new email address.
7. Create a free domain name for the fictitious company at Freenom domain registrar [here](https://www.freenom.com/en/index.html?lang=en).
8. Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that [here](https://www.youtube.com/watch?v=IjcHp94Hq8A)
  
![Screen Shot 2021-07-06 at 12 59 27 PM](https://user-images.githubusercontent.com/44268796/124639367-12df9e80-de5a-11eb-9eb9-a19cf30382d8.png)
  
#### TLS Certificates From Amazon Certificate Manager (ACM)

TLS certificates are required to handle secured connectivity to the Application Load Balancers (ALB).
- Navigate to AWS ACM
- Request a public wildcard certificate for the domain name previously registered
- Use DNS to validate the domain name
- Tag the resource
  
![Screen Shot 2021-07-08 at 2 14 30 PM](https://user-images.githubusercontent.com/44268796/124971124-d2695780-dff6-11eb-804e-ad6ba1879657.png)



#### Set Up a Virtual Private Network (VPC)
  
###### Always make reference to the architectural diagram and ensure that the configuration is aligned with it.

1. Create a VPC and also enbale DNS Hostname resolution for the VPC
  
  ![Screen Shot 2021-07-06 at 1 43 05 PM](https://user-images.githubusercontent.com/44268796/124644353-27269a00-de60-11eb-9693-50f6a2913f4f.png)

2. Create subnets as shown in the architecture
  
  ![Screen Shot 2021-07-06 at 1 55 08 PM](https://user-images.githubusercontent.com/44268796/124645734-c8622000-de61-11eb-99f9-b0313e97cce8.png)

  
3. Create a route table and associate it with public subnets
4. Create a route table and associate it with private subnets
  
  ![Screen Shot 2021-07-06 at 2 25 16 PM](https://user-images.githubusercontent.com/44268796/124649058-ffd2cb80-de65-11eb-88c6-25047134b849.png)

5. Create an Internet Gateway and attach it to the VPC
  
  ![Screen Shot 2021-07-06 at 2 27 05 PM](https://user-images.githubusercontent.com/44268796/124649252-3f011c80-de66-11eb-9592-c63a0651d9a9.png)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet). specify 0.0.0.0/0 in the Destination box, and select the internet gateway ID in the Target list
  
  ![Screen Shot 2021-07-06 at 2 33 37 PM](https://user-images.githubusercontent.com/44268796/124649952-29d8bd80-de67-11eb-84ee-df1fe3d6be31.png)

7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts). Add a new route on the private route table to configure destination as 0.0.0.0/0 and target as NAT Gateway.
  
  ![Screen Shot 2021-07-06 at 2 42 07 PM](https://user-images.githubusercontent.com/44268796/124651057-783a8c00-de68-11eb-877e-145460872307.png)

  ![Screen Shot 2021-07-06 at 2 44 00 PM](https://user-images.githubusercontent.com/44268796/124651207-a029ef80-de68-11eb-97ae-94577d356690.png)

9. Create a Security Group for:
  
 - Nginx Servers: Access to Nginx should only be allowed from a  External Application Load balancer (ALB). At this point, a load balancer has not been created, therefore  the rules will be updatedlater. For now, just create it and put some dummy records as a place holder.

- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence,  use the workstation public IP address. To get this information, simply go to terminal and type curl www.canhazip.com
  
- External Application Load Balancer: The External ALB will be available from the Internet and also will allow SSH from the bastion host.
  
- Internal Application Load Balancer: This ALB will only allow HTTP and HTTPS traffic from the Nginx Reverse Proxy Servers.
  
- Webservers: The security group should allow SSH from bastion host, HTTP and HTTPS from the internal ALB only.
  
- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully designed - webservers need to mount the file system and connect to the RDS database, bastion host needs to have SQL access to the RDS to use the MYSQL client
  
  
![Screen Shot 2021-08-02 at 10 26 26 AM](https://user-images.githubusercontent.com/44268796/127877242-73ad9923-023d-4c31-a104-48c6d1e3ce80.png)
  
This project utilizes EFS service and mount filesystems on the Nginx and Webservers to store data.

1. Create an EFS filesystem
2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer. Mount the EFS in both AZ in the same private subnets as the webservers (private-subnet-01 and private-subnet-02)
3. Associate the Security groups created earlier for data layer.
4. Create two separate EFS access points- one for wordpress and one for tooling. (Give it a name and leave all other settings as default). Set the POSIX user and Group ID to root(0) and permissions to 0755
 
![Screen Shot 2021-08-02 at 10 50 12 AM](https://user-images.githubusercontent.com/44268796/127880713-ab84edab-ef52-4883-806d-d91593fe6508.png)
  
![Screen Shot 2021-08-02 at 10 53 49 AM](https://user-images.githubusercontent.com/44268796/127881251-37030c1f-91a6-40a6-b8d5-3e696c5d85ad.png)

![Screen Shot 2021-08-02 at 10 56 32 AM](https://user-images.githubusercontent.com/44268796/127881639-91ce3c68-0dcf-4172-a91c-6b363862aa5e.png)

![Screen Shot 2021-08-02 at 10 57 03 AM](https://user-images.githubusercontent.com/44268796/127881702-5d236ddd-75a7-4c05-ad09-0c82d3d87b7d.png)

  
  
#### Setup RDS
  
##### Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance

- On KMS Console, choose Symmetric and Click Next. Assign an Alias. 
- Select user with admin privileges as the key administrator
- Click Create Key
  
![Screen Shot 2021-08-02 at 11 23 29 AM](https://user-images.githubusercontent.com/44268796/127885682-cc99a306-f915-4fcd-a99e-7b2eb61ad5ba.png)

 
##### Create DB Subnet Group
  
On the RDS Management Console, create a DB subnet group with the two private subnets(10.0.5.0/24 and 10.0.7.0/24) in the two Availability Zones.
  
![Screen Shot 2021-08-02 at 11 28 07 AM](https://user-images.githubusercontent.com/44268796/127886143-c54d8856-d345-405c-89af-bcea1a39ce2d.png)

    
##### Create RDS
- Click on Create Database
- Select MySQL
- Choose Free Tier from the Templates
- Enter a name for the DB 
- Create Master username and passsword
- Select the VPC, select the subnet group that was created earlier and the database security group
- Scroll down to Additional configuration
- Enter initial database name 
- Scroll down and click Create database

![Screen Shot 2021-08-02 at 11 37 54 AM](https://user-images.githubusercontent.com/44268796/127887350-5423fc44-5841-4d5e-af45-fe0d4a5b077a.png)
  
 #### Proceed With Compute Resources

Configure compute resources inside the VPC. The recources related to compute are:

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)
  
Configure the Public Subnets as follows so the instances can be assigned a public IPv4 address:
- In the left navigation pane, choose Subnets.
- Select the public subnet for the VPC. By default, the name created by the VPC wizard is Public subnet.
- Choose Actions, Modify auto-assign IP settings.
- Select the Enable auto-assign public IPv4 address check box, and then choose Save.
  

#### Launch templates 
##### First create three EC2 instance for Nginx server, Bastion server and Webserver.
- Launch 3 EC2 instances (Red Hat Free Tier) with default settings and a security group that allows access from all traffic (0.0.0.0/0)
  
![Screen Shot 2021-08-02 at 11 49 59 AM](https://user-images.githubusercontent.com/44268796/127889000-303a96ca-5ce6-4ab3-9d48-ed0ef465073d.png)

##### Set up the Bastion server:
  
Install the following:
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm 
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 
yum install wget vim python3 telnet htop git mysql net-tools chrony -y 
systemctl start chronyd
systemctl enable chronyd
```
  
##### Set up the Nginx server for AMI installation:
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd
```
Configure Selinux policies with the following. These policies are required for the webserver to function properly:
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1  
```
Install amazon EFS utils for mounting the target on the Elastic file system:
```
git clone https://github.com/aws/efs-utils
cd efs-utils
yum install -y make
yum install -y rpm-build
make rpm 
yum install -y  ./build/amazon-efs-utils*rpm
```
Install self-signed certificate for Nginx instance:
```
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```  
##### Set up of webserver instance for AMI:
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd
```
Configure Selinux policies:
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
Install EFS utils:
```
git clone https://github.com/aws/efs-utils
cd efs-utils
yum install -y make
yum install -y rpm-build
make rpm 
yum install -y  ./build/amazon-efs-utils*rpm
``` 
# Setting up self-signed certificate for the Apache webserver instance:
```
yum install -y mod_ssl
openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt
vi /etc/httpd/conf.d/ssl.conf
``` 

Modify the file with the key name and certificate name: 
![Screen Shot 2021-08-02 at 2 18 13 PM](https://user-images.githubusercontent.com/44268796/127906077-8fb2d831-b81b-4078-9dcb-c1bec12b2496.png)

#### Create AMIs from the Nginx, Bastion and Webserver Instances
  
![Screen Shot 2021-08-02 at 2 44 30 PM](https://user-images.githubusercontent.com/44268796/127908855-55383cd9-4591-46a1-814a-f72315d95286.png)

#### Create three Target Group for Nginx server, WordPress server and Tooling server respectively with the same settings:

1. Select Instances as the target type
2. Ensure the protocol HTTPS on secure TLS port 443
3. Ensure that the health check path is /healthstatus
  
![Screen Shot 2021-08-02 at 2 50 33 PM](https://user-images.githubusercontent.com/44268796/127909457-f7e8e516-0f9e-4844-aa03-5b4ae47f90c8.png)

#### Create Load Balancer using the Target Groups created
  
##### Create the External Application Load Balancer:
- Choose the right VPC
- Choose public subnets 1 and 2
- Select HTTPS protocol
- Select the Nginx target group
- Create the Load Balancer
  
![Screen Shot 2021-08-02 at 2 57 49 PM](https://user-images.githubusercontent.com/44268796/127910189-1aee2cbd-4ad1-4b37-91b4-e550d3c1bd88.png)

  
##### Create the Internal Application Load Balancer:
- Choose the right VPC
- Choose private subnets 1 and 2
- Ensure that it is facing internally
- Select HTTPS protocol
- Select the WordPress Target Group as default target group (Tooling will be configured soon)
- Create the Load Balancer
 
  ![Screen Shot 2021-08-02 at 3 02 53 PM](https://user-images.githubusercontent.com/44268796/127910680-2208427b-27cf-45d5-85d4-a40060e88b2b.png)

 To configure the Internal Load Balancer to forward traffic to Tooling webserver, add a new rule and configure to forward the traffic to Tooling Target Group as below:
  
![Screen Shot 2021-08-02 at 3 07 59 PM](https://user-images.githubusercontent.com/44268796/127911319-0bc4278d-3bff-4719-9951-0da7e0a9c671.png)
  
![Screen Shot 2021-08-02 at 3 10 46 PM](https://user-images.githubusercontent.com/44268796/127911455-ef1fa201-b64d-473e-8ea9-0536d8e8c565.png)
  
#### Create Launch Templates: 

##### Create Bastion Launch Template:

- Make use of the AMI to set up a launch template
- Ensure the Instances are launched into a public subnet 
- Assign appropriate security group
- Create a Network Interface and also enable the "auto-assign public IP" feature
- Configure Userdata to update yum package repository and install Ansible and git
```
#!/bin/bash 
yum install -y mysql 
yum install -y git tmux 
yum install -y ansible
```

##### Create Nginx Launch Template:
  
- Follow same steps as above; choose the Nginx AMI; Nginx security group, enable auto-assign IP, and configure userdata:
```
#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/tshenoy19/project15-config.git
mv /project15-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /project15-config
```
The above repository in the userdata was forked from https://github.com/Livingstone95/ACS-project-config.git and renamed. 
  
The reverse.conf file was modified to use the custom server name and internal load balancer DNS name.
  

  

  



  

  

  





  


  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
