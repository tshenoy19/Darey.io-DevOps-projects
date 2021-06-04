
### Ansible Configuration Management - Automate Project 7 to 10

This project will make use of Ansible Configuration Management to automate routine server configuration tasks. 
The project will focus on developing Ansible scripts to simulate the use of a Jump box/Bastion host to access the Web Servers.

#### Task:

- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

#### Step 1 - Install and configure Ansible on EC2 Instance

1. Update Name tag on the Jenkins EC2 Instance to Jenkins-Ansible. This server will be used to run the playbooks.

![Screen Shot 2021-06-04 at 9 55 33 AM](https://user-images.githubusercontent.com/44268796/120812516-0abed700-c51b-11eb-9678-63b2b6ca8f42.png)


3. In the GitHub account, create a new repository and name it ansible-config-mgt.
4. Install Ansible

```
sudo apt update

sudo apt install ansible
```
