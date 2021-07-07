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

#### Set Up a Virtual Private Network (VPC)
  
###### Always make reference to the architectural diagram and ensure that the configuration is aligned with it.

1. Create a VPC
  
  ![Screen Shot 2021-07-06 at 1 43 05 PM](https://user-images.githubusercontent.com/44268796/124644353-27269a00-de60-11eb-9693-50f6a2913f4f.png)

2. Create subnets as shown in the architecture
  
  ![Screen Shot 2021-07-06 at 1 55 08 PM](https://user-images.githubusercontent.com/44268796/124645734-c8622000-de61-11eb-99f9-b0313e97cce8.png)

  
3. Create a route table and associate it with public subnets
4. Create a route table and associate it with private subnets
  
  ![Screen Shot 2021-07-06 at 2 25 16 PM](https://user-images.githubusercontent.com/44268796/124649058-ffd2cb80-de65-11eb-88c6-25047134b849.png)

5. Create an Internet Gateway
  
  ![Screen Shot 2021-07-06 at 2 27 05 PM](https://user-images.githubusercontent.com/44268796/124649252-3f011c80-de66-11eb-9592-c63a0651d9a9.png)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet). specify 0.0.0.0/0 in the Destination box, and select the internet gateway ID in the Target list
  
  ![Screen Shot 2021-07-06 at 2 33 37 PM](https://user-images.githubusercontent.com/44268796/124649952-29d8bd80-de67-11eb-84ee-df1fe3d6be31.png)

7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts). Add a new route on the private route table to configure destination as 0.0.0.0/0 and target as NAT Gateway.
  
  ![Screen Shot 2021-07-06 at 2 42 07 PM](https://user-images.githubusercontent.com/44268796/124651057-783a8c00-de68-11eb-877e-145460872307.png)

  ![Screen Shot 2021-07-06 at 2 44 00 PM](https://user-images.githubusercontent.com/44268796/124651207-a029ef80-de68-11eb-97ae-94577d356690.png)

9. Create a Security Group for:
  
 - Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, a load balancer has not been created, therefore  the rules will be updatedlater. For now, just create it and put some dummy records as a place holder.

- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence,  use the workstation public IP address. To get this information, simply go to terminal and type curl www.canhazip.com
  
- Application Load Balancer: ALB will be available from the Internet
  
- Webservers: Access to Webservers should only be allowed from the Nginx servers. Since the servers have not been created yet, just put some dummy records as a place holder for now.
  
- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully designed - only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.
  
![Screen Shot 2021-07-06 at 2 59 35 PM](https://user-images.githubusercontent.com/44268796/124653018-cc467000-de6a-11eb-915b-94cb0466e022.png)
  
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
  
##### Create an AMI out of the EC2 instance
  
![Screen Shot 2021-07-06 at 4 25 16 PM](https://user-images.githubusercontent.com/44268796/124662376-c22a6e80-de76-11eb-9a2e-160ca195ce3e.png)
  
##### Prepare Launch Template For Nginx (One Per Subnet)
  
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install nginx:
```
#!/bin/bash
yum update -y
yum install -y nginx
systemctl start nginx
systemctl enable nginx
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
  




  




  


  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
