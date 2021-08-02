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
  
#### Set Up Compute Resources for Nginx
  
##### Provision EC2 Instances for Nginx

Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)
  
Ensure that it has the following software installed:
- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
  
A shell script with the below installation commands would be useful as the same software will be needed to be installed on many servers in this project. 
  
```
sudo yum install python3
sudo dnf install chrony
sudo systemctl start chronyd
sudo systemctl status chronyd
sudo systemctl enable chronyd
sudo yum install net-tools
sudo yum install vim
sudo yum install wget
sudo yum install telnet
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf update
sudo rpm -qa | grep epel
sudo dnf --disablerepo="*" --enablerepo="epel" list available
sudo dnf install htop
```
  
##### Create an AMI out of the EC2 instance
  
![Screen Shot 2021-07-06 at 4 25 16 PM](https://user-images.githubusercontent.com/44268796/124662376-c22a6e80-de76-11eb-9a2e-160ca195ce3e.png)
  
##### Prepare Launch Template For Nginx (One Per Subnet)
  
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install nginx:
```
#!/bin/bash
yum install -y  https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install -y nginx
systemctl start nginx
systemctl enable nginx
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php7.4-zip php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
yum install -y git
git clone https://github.com/tshenoy19/project15-conf.git
mv project15-conf/reverse /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse
rm -rf reverse
systemctl restart nginx
```
  
![Screen Shot 2021-07-06 at 4 30 26 PM](https://user-images.githubusercontent.com/44268796/124662967-7a581700-de77-11eb-8636-5e318091191c.png)
  
![Screen Shot 2021-07-06 at 4 41 16 PM](https://user-images.githubusercontent.com/44268796/124664130-fd2da180-de78-11eb-93b2-7398da8cb747.png)
  
##### Configure Target Groups

1. Select Instances as the target type
2. Ensure the protocol HTTPS on secure TLS port 443
3. Ensure that the health check path is /healthstatus
4. Register Nginx Instances as targets
5. Ensure that health check passes for the target group
  
![Screen Shot 2021-07-06 at 4 44 25 PM](https://user-images.githubusercontent.com/44268796/124664427-6f9e8180-de79-11eb-8a00-85f3d96942b0.png)

##### Configure Autoscaling For Nginx

1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the Auto Scaling Group (ASG)
5. Select the target group you created before
6. Ensure that the health checks are for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11.Ensure there is an SNS topic to send scaling notifications
  
  
![Screen Shot 2021-07-07 at 10 21 57 AM](https://user-images.githubusercontent.com/44268796/124776111-2dbf1b00-df0d-11eb-8ccb-66e2adee397b.png)

![Screen Shot 2021-07-07 at 10 24 46 AM](https://user-images.githubusercontent.com/44268796/124776642-9ad2b080-df0d-11eb-86e5-44278d07c490.png)
  
  
#### Set Up Compute Resources for Bastion

##### Provision the EC2 Instances for Bastion

1. Create an EC2 Instance based on RHEL 8 Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
2. Ensure that it has the following software installed
- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
3. Associate an Elastic IP with each of the Bastion EC2 Instances
4. Create an AMI out of the EC2 instance

![Screen Shot 2021-07-07 at 11 13 57 AM](https://user-images.githubusercontent.com/44268796/124784962-6f06f900-df14-11eb-87dc-2500625514a5.png)
  

##### Prepare Launch Template For Bastion (One per subnet)
- Make use of the AMI to set up a launch template
- Ensure the Instances are launched into a public subnet
- Assign appropriate security group
- Configure Userdata to update yum package repository and install Ansible and git
```
#!/bin/bash
yum install -y  https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install -y mysql
yum install -y git
yum install -y ansible
```

![Screen Shot 2021-07-08 at 11 29 49 AM](https://user-images.githubusercontent.com/44268796/124949715-d178fb80-dfdf-11eb-9004-6f38a175c803.png)

##### Configure Target Groups
- Select Instances as the target type
- Ensure the protocol is TCP on port 22
- Register Bastion Instances as targets
- Ensure that health check passes for the target group
  
![Screen Shot 2021-07-08 at 11 33 57 AM](https://user-images.githubusercontent.com/44268796/124950446-6b40a880-dfe0-11eb-91f0-8f6a5a939819.png)
  
##### Configure Autoscaling For Bastion
1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the target group you created before
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications
  
![Screen Shot 2021-07-07 at 11 30 33 AM](https://user-images.githubusercontent.com/44268796/124787753-c017ec80-df16-11eb-9fa6-9b55f1868ea3.png)
  
#### Set Up Compute Resources for Webservers
  
##### Provision the EC2 Instances for Webservers

We will need to create 2 separate launch templates for both the WordPress and Tooling websites

Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).
Ensure that it has the following software installed:
- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
- php

Create an AMI out of the EC2 instance
  
![Screen Shot 2021-07-08 at 2 03 57 PM](https://user-images.githubusercontent.com/44268796/124969999-6508f700-dff5-11eb-8265-dc84705a28f2.png)

  
##### Prepare Launch Template For Webservers (One per subnet)
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)

The user data for the WordPress webserver launch template is below:
```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum install -y git
yum install -y mysql
yum install -y php php-fpm php-mysqlnd
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
systemctl restart httpd
```


Repeat the same steps for launch template for the Tooling server with the user data below:
```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum install -y git
yum install -y mysql
yum install -y php php-{mysqlnd,cli,gd,common,mbstring,fpm,json}
systemctl restart httpd
```
  
![Screen Shot 2021-07-08 at 2 11 30 PM](https://user-images.githubusercontent.com/44268796/124970822-671f8580-dff6-11eb-97d6-0fc531ae5e2e.png)
  
##### Create two separate Target Groups for WordPress servers and Tooling servers
  
![Screen Shot 2021-07-08 at 2 34 03 PM](https://user-images.githubusercontent.com/44268796/124973545-a9969180-dff9-11eb-94c5-a630738e8494.png)

 
##### Application Load Balancer To Route Traffic To Web Servers

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

- Create an Internal ALB
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM
- Select Security Group
- Select webserver Instances as the target group
- Ensure that health check passes for the target group
NOTE: This process must be repeated for both WordPress and Tooling websites.
  
![Screen Shot 2021-07-08 at 2 46 16 PM](https://user-images.githubusercontent.com/44268796/124974832-4443a000-dffb-11eb-9eb8-1d9ed00cd1d7.png) 
  
![Screen Shot 2021-07-08 at 2 45 32 PM](https://user-images.githubusercontent.com/44268796/124974743-27a76800-dffb-11eb-97cf-7be7ace9b622.png)

#### Setup EFS
  
This project utilizes EFS service and mount filesystems on the Nginx and Webservers to store data.

1. Create an EFS filesystem
2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
3. Associate the Security groups created earlier for data layer.
4. Create an EFS access point. (Give it a name and leave all other settings as default)
 
![Screen Shot 2021-07-08 at 2 57 17 PM](https://user-images.githubusercontent.com/44268796/124976085-cbddde80-dffc-11eb-9f8e-00af28c0a1bd.png)

![Screen Shot 2021-07-08 at 2 58 30 PM](https://user-images.githubusercontent.com/44268796/124976209-f760c900-dffc-11eb-9769-0570cbb71eb1.png)

#### Setup RDS
  
##### Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance

- On IAM, create two AWS Service roles for RDS administrative privileges- A Service Role for RDS and another Role for RDS monitoring
- On KMS Console, choose Symmetric and Click Next. Assign an Alias. 
- Select the two roles to assign administrative RDS privileges
- Click Create Key
  
![Screen Shot 2021-07-09 at 11 33 52 AM](https://user-images.githubusercontent.com/44268796/125103360-457fd600-e0aa-11eb-969b-92c952b64d62.png)

![Screen Shot 2021-07-09 at 11 34 37 AM](https://user-images.githubusercontent.com/44268796/125103384-4a448a00-e0aa-11eb-8e96-d42bc9af7c02.png)
 
##### Create DB Subnet Group
  
On the RDS Management Console, create a DB subnet group with two private subnets in the two Availability Zones
  
![Screen Shot 2021-07-09 at 10 29 12 AM](https://user-images.githubusercontent.com/44268796/125093820-b4583180-e0a0-11eb-8fed-3b505be60066.png)
  
##### Create RDS
- Click on Create Database
- Select MySQL
- Choose Dev/Test from the Templates
- Enter a name for the DB 
- Create Master username and passsword
- Choose the smallest possible instance to keep the costs low (db.t3.micro etc.,)
- Select "Do not create a standby instance" option
- Select the VPC, select the subnet group that was created earlier and the database security group
- Scroll down to Additional configuration
- Enter initial database name (but i'll personally recommend you connect to it from your webservers and create required databases)
- Leave everything else, scroll down to Encryption and select the KMS key you created
- Scroll down and click Create database

![Screen Shot 2021-07-09 at 11 43 27 AM](https://user-images.githubusercontent.com/44268796/125104000-ea9aae80-e0aa-11eb-8b00-6a12d9e0a81a.png)

  

  
  

  




  




  


  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
