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
5. Move the DevOps account into the Dev OU.
6. Login to the newly created AWS account using the new email address.
7. Create a free domain name for the fictitious company at Freenom domain registrar [here](https://www.freenom.com/en/index.html?lang=en).
8. Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that [here](https://www.youtube.com/watch?v=IjcHp94Hq8A)

  
