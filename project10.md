
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



![Screen Shot 2021-06-02 at 7 08 43 AM](https://user-images.githubusercontent.com/44268796/120470309-60f01680-c371-11eb-90c1-a668cca99f8b.png)


###### Install Nginx

```
sudo apt update
sudo apt install nginx
```


Configure /etc/hosts on the Nginx Load Balancer server to add the web servers:
```
sudo nano /etc/hosts
```

![Screen Shot 2021-06-02 at 7 15 36 AM](https://user-images.githubusercontent.com/44268796/120471201-6c900d00-c372-11eb-9e37-4ffabd12ee08.png)

Next step is to configure the Load Balancer to point traffic to the DNS names of the two webservers:


```
sudo nano /etc/nginx/nginx.conf

#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
Restart Nginx and check status:
```
sudo systemctl restart nginx
sudo systemctl status nginx
```


![Screen Shot 2021-06-02 at 7 24 52 AM](https://user-images.githubusercontent.com/44268796/120472236-a1e92a80-c373-11eb-9a6d-65124ea928e0.png)


![Screen Shot 2021-06-02 at 9 42 49 AM](https://user-images.githubusercontent.com/44268796/120491021-e6ca8c80-c386-11eb-9f85-31eea733a6aa.png)


#### Register a new domain name and configure secured connection using SSL/TLS certificates

To obtain a valid SSL certificate, a domain name needs to be registered. The following steps were taken:

- Registered a domain name with Godaddy.com

![Screen Shot 2021-06-02 at 10 08 00 AM](https://user-images.githubusercontent.com/44268796/120495135-6c037080-c38a-11eb-8280-8bfa87dfe6f6.png)

- Assigned an Elastic IP to the Nginx LB server and associated the domain name with this Elastic IP

![Screen Shot 2021-06-02 at 10 07 22 AM](https://user-images.githubusercontent.com/44268796/120495043-54c48300-c38a-11eb-9b37-c2fb717be48f.png)

- Updated a record in the domain name registrar to point to the Nginx LB using Elastic IP address

![Screen Shot 2021-06-02 at 10 16 18 AM](https://user-images.githubusercontent.com/44268796/120496462-943f9f00-c38b-11eb-9bd5-4a05c739e3bd.png)

###### Configure Nginx to recognize your new domain name

Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com.
Check that the tooling application can be reached from the browser using new domain name using HTTP protocol - http://<your-domain-name.com>

###### Install certbot and request for an SSL/TLS certificate

Before installing certbot, ensure that the snapd service is active and running

```
sudo apt update 
sudo apt install snapd 
sudo systemctl status snapd
```

![Screen Shot 2021-06-02 at 10 38 18 AM](https://user-images.githubusercontent.com/44268796/120500190-ab33c080-c38e-11eb-9cf8-87b640367b45.png)

Install certbot
```
sudo snap install --classic certbot
```

![Screen Shot 2021-06-02 at 10 39 51 AM](https://user-images.githubusercontent.com/44268796/120500475-de764f80-c38e-11eb-9700-601f024b347a.png)


Request the SSL certificate. Choose the domain that the certificate is to be issued for, domain name will be looked up from nginx.conf.

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

![Screen Shot 2021-06-02 at 10 44 42 AM](https://user-images.githubusercontent.com/44268796/120501356-8be96300-c38f-11eb-956a-ae5cb756c050.png)

The web solution should now be securely accessible at https://<your-domain-name.com>

![Screen Shot 2021-06-02 at 10 46 08 AM](https://user-images.githubusercontent.com/44268796/120501616-bf2bf200-c38f-11eb-8969-ad43213d04f1.png)

Access the website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in the browser’s search string. Click on the padlock icon to find the details of the certificate issued for the website.



![Screen Shot 2021-06-02 at 10 49 01 AM](https://user-images.githubusercontent.com/44268796/120502138-2cd81e00-c390-11eb-8819-9e048dba30db.png)

###### Set up periodical renewal of your SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently. Test the renewal command in dry-run mode:

```
sudo certbot renew --dry-run
```

![Screen Shot 2021-06-02 at 10 54 34 AM](https://user-images.githubusercontent.com/44268796/120503012-ecc56b00-c390-11eb-86d0-a5494bd3bb4d.png)


###### Setting up a cronjob for automatic renewal of the SSL certificate

Best pracice is to have a scheduled job that to run renew command periodically. Configure a cronjob to run the command twice a day.
To do so, edit the crontab file with the following command:

```
crontab -e
```

Choose nano as the editor and add the following line:
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

Credits:
https://darey.io/


























