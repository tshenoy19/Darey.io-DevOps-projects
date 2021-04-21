
# WEB STACK IMPLEMENTATION (LEMP STACK)

## Step 0 - Preparing prerequisites

Sign in to AWS free tier account and create a new EC2 Instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image.
SSH into the instance.

## Step 1 – Installing the Nginx Web Server
```bash
sudo apt update
sudo apt install nginx
sudo systemctl status nginx
```

We have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80.

Test if the Nginx server can respond to requests from the Internet. Open a web browser of your choice and try to access following url:
```
http://<Public-IP-Address>:80
```

If you see following page, then your web server is now correctly installed and accessible through your firewall.

