
### Load Balancer Solution With Nginx and SSL/TLS

This project uses the same infrastructure as project 8 but in this case, the load balancer solution will be configured with Nginx instead of Apache. 

In order to ensure that the application is secure in transit, SSL/TLS digital certificates will be used to identify and validate the website. In this project, the website will be registered with LetsEnrcypt Certificate Authority. To automate certificate issuance, a shell client recommended by LetsEncrypt - cetrbot will be used. 


#### Task

This project consists of two parts:

Configure Nginx as a Load Balancer
Register a new domain name and configure secured connection using SSL/TLS certificates

The target architecture is as follows:


![Screen Shot 2021-06-02 at 6 53 30 AM](https://user-images.githubusercontent.com/44268796/120468457-40bf5800-c36f-11eb-8c2a-e49de6e84ae8.png)


#### Part 1 - Configure Nginx As A Load Balancer

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections)

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

###### Install Nginx


