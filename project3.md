
# Simple To-Do application on MERN Web Stack

## Step 0 - Preparing prerequisites
 On AWS, create a new EC2 Instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image
 
## Step 1: Backend Configuration

Update Ubuntu and Upgrade Ubuntu
```bash
sudo apt update
sudo apt upgrade

```

Get location of Node.js software from Ubuntu repositories
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Install Node.js and npm with the following command:
```
sudo apt-get install -y nodejs
```
### Application Code Setup

Create a directory for the To-Do Project:
```
mkdir Todo
```
Change Directory:
```
cd Todo
```
To create a package.json file, run the command:
```
npm init
```























