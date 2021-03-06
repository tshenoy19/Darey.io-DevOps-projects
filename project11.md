
### Ansible Configuration Management - Automate Project 7 to 10

This project will make use of Ansible Configuration Management to automate routine server configuration tasks. 
The project will focus on developing Ansible scripts to simulate the use of a Jump box/Bastion host to access the Web Servers.

#### Task:

- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

#### Step 1 - Install and configure Ansible on EC2 Instance:

##### 1. Update Name tag on the Jenkins EC2 Instance to Jenkins-Ansible. This server will be used to run the playbooks.

![Screen Shot 2021-06-04 at 9 55 33 AM](https://user-images.githubusercontent.com/44268796/120812516-0abed700-c51b-11eb-9678-63b2b6ca8f42.png)


##### 2. In the GitHub account, create a new repository and name it ansible-config-mgt.

![Screen Shot 2021-06-04 at 10 02 36 AM](https://user-images.githubusercontent.com/44268796/120813563-ff1fe000-c51b-11eb-8b2f-9b54f35bd398.png)


##### 3. Install Ansible

```
sudo apt update

sudo apt install ansible

ansible --version
```
![Screen Shot 2021-06-04 at 10 00 45 AM](https://user-images.githubusercontent.com/44268796/120813289-bf58f880-c51b-11eb-9df8-cd53d11ff458.png)


##### 4. Configure Jenkins build job to save the repository content every time any changes are made:

- Create a new Freestyle project ansible in Jenkins and point it to the ‘ansible-config-mgt’ repository.

![Screen Shot 2021-06-04 at 10 12 24 AM](https://user-images.githubusercontent.com/44268796/120814936-5d998e00-c51d-11eb-8c43-944d1872e9ec.png)

![Screen Shot 2021-06-04 at 10 16 04 AM](https://user-images.githubusercontent.com/44268796/120815510-e1ec1100-c51d-11eb-8f37-d4b35a3f1e83.png)

- Configure Webhook in GitHub and set webhook to trigger ansible build.

![Screen Shot 2021-06-04 at 10 17 42 AM](https://user-images.githubusercontent.com/44268796/120815732-1b248100-c51e-11eb-92e6-7828151b0756.png)

- Configure a Post-build job to save all (**)files using the same steps as Project 9.
- 
- Test the setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder:

```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
###### The build failed many times. After trying many different configuration changes, I was finally able to get the build to be successful by changing the branch specifier to */main instead of */master. Another change was that I had to update the Jenkins URL address with the new public IP address of the server as I had restarted the server since last time. Assigning an Elastic IP address to the Jenkins server would also fix this issue. 

![Screen Shot 2021-06-04 at 10 40 38 AM](https://user-images.githubusercontent.com/44268796/120819250-612f1400-c521-11eb-89ee-81ea4e991837.png)

![Screen Shot 2021-06-04 at 10 42 13 AM](https://user-images.githubusercontent.com/44268796/120819393-87ed4a80-c521-11eb-9f85-66d2be3a11fe.png)


The architecture of the setup so far looks like the diagram below:

![Screen Shot 2021-06-04 at 10 45 12 AM](https://user-images.githubusercontent.com/44268796/120819865-fd591b00-c521-11eb-9f89-d45b6003077a.png)


#### Step 2 - Prepare the development environment using Visual Studio Code

Next step is to install a versatile Integrated development environment (IDE) or Source-code Editor. The IDE used in this project is Visual Studio Code (VSC). Configure it and connect it to the GitHub respository created for this project (https://code.visualstudio.com/docs/editor/github).


![Screen Shot 2021-06-05 at 11 09 36 AM](https://user-images.githubusercontent.com/44268796/120896210-8d649680-c5ee-11eb-8c62-44493b2ffc8b.png)

#### Step 3 - Begin Ansible Development

In the ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature - ansible-newfeature. 

![Screen Shot 2021-06-05 at 11 33 00 AM](https://user-images.githubusercontent.com/44268796/120896947-c9e5c180-c5f1-11eb-8755-20d12ddc5a1b.png)


1. Checkout the newly created feature branch to your local machine and start building your code and directory structure

![Screen Shot 2021-06-05 at 11 30 41 AM](https://user-images.githubusercontent.com/44268796/120896878-78d5cd80-c5f1-11eb-9354-ec1f11ccaeda.png)


2. Create a directory and name it playbooks - it will be used to store all your playbook files. Also, create a directory and name it inventory - it will be used to keep your hosts organised.

![Screen Shot 2021-06-05 at 11 37 01 AM](https://user-images.githubusercontent.com/44268796/120897045-5a240680-c5f2-11eb-9f4a-2528cded1678.png)


3. Within the playbooks folder, create your first playbook, and name it common.yml.
```
cd playbooks
touch common.yml
```
4.  Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
```
cd inventory
touch dev.yml staging.yml uat.yml prod.yml
```


![Screen Shot 2021-06-05 at 11 41 39 AM](https://user-images.githubusercontent.com/44268796/120897237-0ebe2800-c5f3-11eb-8be8-9919da91b7f0.png)


![Screen Shot 2021-06-05 at 11 41 52 AM](https://user-images.githubusercontent.com/44268796/120897269-3a411280-c5f3-11eb-8cfc-7d25f8994638.png)

#### Step 4 - Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. The intention is to execute Linux commands on remote hosts and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the inventory/dev file to start configuring the development servers. Ensure to replace the IP addresses with the actual IP addresses of the setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this copy the private (.pem) key to the server. Change permissions to the private key with chmod 400 key.pem, otherwise EC2 will not accept the key. Import the key into ssh-agent:

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

##### Blocker: It was a huge learning experience to figure out how to transfer files from local machine to EC2 instance and also to set up the permissions and key pair transfer to the Ansible master in order to be able to connect with the host servers. I saved a copy of the pem file in the ansible-config-mgt folder and added the path to the inventory file. 

![Screen Shot 2021-06-09 at 10 07 44 AM](https://user-images.githubusercontent.com/44268796/121371086-41c22d80-c90b-11eb-8ae8-fae4fee70ee0.png)


In this setup, the Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

Update the inventory/dev.yml file with this snippet of code:
```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' ansible_ssh_private_key_file=<path-to-.pem-private-key>

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu' ansible_ssh_private_key_file=<path-to-.pem-private-key>
```

![Screen Shot 2021-06-09 at 10 20 13 AM](https://user-images.githubusercontent.com/44268796/121372426-49360680-c90c-11eb-807f-8560b42e6531.png)


#### Step 5 - Create a Common Playbook

The common.yml playbook will contain configuration code for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update the playbooks/common.yml file with following code:

```yml
---
- name: update web, nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB and db server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    apt:
      name: wireshark
      state: latest
  ```

![Screen Shot 2021-06-09 at 10 23 34 AM](https://user-images.githubusercontent.com/44268796/121373097-cceff300-c90c-11eb-9659-d6bb5a44366c.png)

This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on the RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

#### Step 6 - Update GIT with the latest code

Now all of the directories and files live on the machine and the changes need to be pushed to GitHub.

The changes have been made in a separate branch, so a pull request needs to be raised, the branch needs to be peer reviewed and merged to the master branch.

Commit the code into GitHub:
```
git status

git add <selected files>

git commit -m "commit message"
```

![Screen Shot 2021-06-09 at 10 32 16 AM](https://user-images.githubusercontent.com/44268796/121374642-0ecd6900-c90e-11eb-9ed8-889df48efac5.png)

![Screen Shot 2021-06-09 at 10 32 51 AM](https://user-images.githubusercontent.com/44268796/121374650-10972c80-c90e-11eb-9e6b-6412e102e4be.png)


##### Integrating the code into the Master branch after reviewing it

1. Create a Pull request (PR).
2. Review the code and configuration from a different perspective.
3. If everything looks okay, merge the code to the master branch.
4. Head back on the terminal, checkout from the feature branch into the master, and pull down the latest changes.


![Screen Shot 2021-06-09 at 10 42 34 AM](https://user-images.githubusercontent.com/44268796/121376309-69b39000-c90f-11eb-88c0-2335992cbe2e.png)

![Screen Shot 2021-06-09 at 11 03 06 AM](https://user-images.githubusercontent.com/44268796/121379712-4c33f580-c912-11eb-8ac6-8484e0f70fb2.png)

With the update to GitHub repository, Jenkins automatically triggered a new build and all the changes were updated in the workspace.

![Screen Shot 2021-06-09 at 11 06 28 AM](https://user-images.githubusercontent.com/44268796/121380309-c3698980-c912-11eb-8da5-04ae27b6b0ce.png)


Run the playbook to execute the commands on the host servers:

```
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

![Screen Shot 2021-06-09 at 11 20 01 AM](https://user-images.githubusercontent.com/44268796/121382477-a5048d80-c914-11eb-9e42-d3db7713a902.png)


The architecture at the end of this project looks like the diagram below:

![Screen Shot 2021-06-09 at 11 22 28 AM](https://user-images.githubusercontent.com/44268796/121383171-3e33a400-c915-11eb-8227-3d3c457a865c.png)


Additional Ansible Exercises:

- Create a directory and a file inside it
- Change timezone on all servers
- Run some shell script

The common.yml playbook was updated with additonal instructions to create a directory called 'test-directory' and also change the timezone to Asia/Tokyo on all servers.


![Screen Shot 2021-06-09 at 3 41 15 PM](https://user-images.githubusercontent.com/44268796/121419191-b1e7a800-c939-11eb-8f56-e3010bb157fc.png)

![Screen Shot 2021-06-09 at 3 42 43 PM](https://user-images.githubusercontent.com/44268796/121419202-b3b16b80-c939-11eb-99f7-a3270822a102.png)




Credits: (https://darey.io/)































