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
5. Create an Internet Gateway
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)
7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
9. Create a Security Group for:
 - Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, a load balancer has not been created, therefore  the rules will be updatedlater. For now, just create it and put some dummy records as a place holder.
- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence,  use the workstation public IP address. To get this information, simply go to terminal and type curl www.canhazip.com
- Application Load Balancer: ALB will be available from the Internet
- Webservers: Access to Webservers should only be allowed from the Nginx servers. Since the servers have not been created yet, just put some dummy records as a place holder for now.
- Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully designed - only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
