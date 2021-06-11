
### Ansible Refactoring & Static Assignments (Imports and Roles)

This project builds upon the previous project by making improvements of the code and working with the ansible-config-mgt repository. The tasks in this project include refactoring the code, creating assignments and learning to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook - it allows to organize tasks and reuse them when needed.

For better understanding or Ansible artifacts re-use, refer to [ansible docs](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html)

##### Code Refactoring

Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

In this project, the primary objective is to improve the Ansible code by reorganizing things around a little bit but the overall state of the infrastructure remains the same.

##### Step 1 - Jenkins job enhancement

With the configuration so far, every change made creates a separate directory on Jenkins, which can get tedious and also consume more space. 
In this step, a new Jenkins job will be created to enhance the configuration.

1. On the jenkins-ansible server, create a new directory called 'ansible-config-artifact'. This directory will store the artifacts from all subsequent builds. 
```
sudo mkdir /home/ubuntu/ansible-config-artifact
```
Change permissions to this directory, so Jenkins could save files there 
```
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
2. On the Jenkins web console, install the Copy Artifact plugin without restarting Jenkins

![Screen Shot 2021-06-11 at 10 06 01 AM](https://user-images.githubusercontent.com/44268796/121699263-c5f7ea80-ca9c-11eb-9a25-80a315525965.png)

3. Create a Freestyle project and name it 'save_artifacts'. Configure the project to be triggered by completion of the existing ansible project.

![Screen Shot 2021-06-11 at 10 12 09 AM](https://user-images.githubusercontent.com/44268796/121700024-7cf46600-ca9d-11eb-9c69-d2b476329af7.png)

The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.


![Screen Shot 2021-06-11 at 10 16 26 AM](https://user-images.githubusercontent.com/44268796/121700616-16237c80-ca9e-11eb-81b2-45425eb12a39.png)


![Screen Shot 2021-06-11 at 10 17 06 AM](https://user-images.githubusercontent.com/44268796/121700722-2f2c2d80-ca9e-11eb-8f95-c2868f3db096.png)

Test the set up by making some change in README.MD file inside the ansible-config-mgt repository (right inside main branch).
If both Jenkins jobs have completed one after another - all the files will be inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to the main branch.

![Screen Shot 2021-06-11 at 10 35 44 AM](https://user-images.githubusercontent.com/44268796/121703419-cbefca80-caa0-11eb-90a0-e6d1a478710c.png)

![Screen Shot 2021-06-11 at 10 35 07 AM](https://user-images.githubusercontent.com/44268796/121703327-b37fb000-caa0-11eb-8773-045a3be1825b.png)

Now the Jenkins pipeline is more neat and clean.

##### Refactor Ansible code by importing other playbooks into site.yml

Before refactoring the code, checkout to a new branch from the main/master branch and name it 'refactor'. Ensure that all the directories and files are present.

![Screen Shot 2021-06-11 at 10 41 42 AM](https://user-images.githubusercontent.com/44268796/121704284-9eefe780-caa1-11eb-9021-bab17a892859.png)

In Project 11, all the tasks were written in a single playbook common.yml, but it is pretty simple set of instructions for only 2 types of OS. However, if there are many more tasks, this playbook will need to be applied to different servers with different requirements. This means that the entire playbook will need to be reviewed to find the tasks that might be applicable and also to make sure new tasks are added to meet the requirements of a larger infrastructure. This could get very time consuming and would not be appreciated by other members of the DevOps team. It would be difficult to use the playbook.

Hence, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

To see the code re-use in action, other playbooks will be imported in this step.

- Within playbooks folder, create a new file and name it site.yml - This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed including common.yml that was created previously. 

- Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of the work. It is not an Ansible-specific concept so organization of the playbooks is entirely up to the developer.

The directory structure should look like this:


![Screen Shot 2021-06-11 at 11 01 18 AM](https://user-images.githubusercontent.com/44268796/121707235-5be34380-caa4-11eb-9492-fef97f6f5649.png)

- Move common.yml file into the newly created static-assignments folder. Inside site.yml file, import common.yml playbook with the following code:
- 
```yml
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

The code above uses built in import_playbook Ansible module.

- Run ansible-playbook command against the dev environment

Since some new tasks need to be applied to the dev servers and wireshark is already installed - go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.

```yml
---
- name: update web and nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB and db server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
      
 ```

Update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:

```
sudo ansible-playbook -i /home/ubuntu/ansible/ansible-config-mgt/inventory/dev.yml /home/ubuntu/ansible/ansible-config-mgt/playbooks/site.yml
```

![Screen Shot 2021-06-11 at 11 27 02 AM](https://user-images.githubusercontent.com/44268796/121710826-f4c78e00-caa7-11eb-8e70-9ec0f14169a8.png)

##### Step 3 - Configure UAT Webservers with a role ‘Webserver’

At this point, the dev environment is nice and clean. In order to keep it that way, configure 2 new Web Servers as uat. Tasks could be written to configure Web Servers in the same playbook, but it would be messy. Instead, it would be best to use a dedicated role to make the configuration reusable.

Launch 2 fresh EC2 instances using RHEL 8 image, which will serve as the uat servers, so name them accordingly - Web1-UAT and Web2-UAT.

To create a role, create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (create roles directory upfront)
```
mkdir roles
cd roles
ansible-galaxy init webserver
```

![Screen Shot 2021-06-11 at 1 56 20 PM](https://user-images.githubusercontent.com/44268796/121729754-d0c27780-cabc-11eb-8e8c-ac953de2f34b.png)

The entire folder structure should look like below. For this project, tests, files, and vars folders can be removed from the roles/ directory.
```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
   ```
   
   After deleting the above folders, the directory structure should look like this:
   ```
   └── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```

- Update the inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of the 2 UAT Web servers as follows:

```
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
```
In /etc/ansible/ansible.cfg file, uncomment roles_path string and provide a full path to the roles directory roles_path = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

1. Install and configure Apache (httpd service)
2. Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
3. Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
4. Make sure httpd service is started
  
 The main.yml file may consist of following tasks:
 ```yml
  ---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
  
  ![Screen Shot 2021-06-11 at 2 15 18 PM](https://user-images.githubusercontent.com/44268796/121731852-7545b900-cabf-11eb-94bd-45e95c8051c3.png)
  
  
#### Step 4 - Reference ‘Webserver’ role
  
Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where the role will be referenced.

```yml
  ---
- hosts: uat-webservers
  roles:
     - webserver
```
  
The entry point to the ansible configuration is the site.yml file. Therefore, the uat-webservers.yml role should be referenced inside site.yml.

```yml
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```
  
  
##### Step 5 - Commit & Test
Commit the changes, create a Pull Request and merge them to master branch, ensure that webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to the Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.
  
![Screen Shot 2021-06-11 at 2 34 37 PM](https://user-images.githubusercontent.com/44268796/121733971-277e8000-cac2-11eb-8c32-da688112f2b2.png)
  


Now run the playbook against the uat inventory:
  
```yml
sudo ansible-playbook -i /home/ubuntu/ansible/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible/ansible-config-mgt/playbooks/site.yml
```
  
![Screen Shot 2021-06-11 at 2 56 03 PM](https://user-images.githubusercontent.com/44268796/121736149-2dc22b80-cac5-11eb-9f74-e23038e836b0.png)  
  
  
  
Both the UAT Web servers are configured now and can be reached from the browser:
```
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
```
or
```
http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
```
  
![Screen Shot 2021-06-11 at 2 45 34 PM](https://user-images.githubusercontent.com/44268796/121735123-b049eb80-cac3-11eb-8818-6d04bdb78811.png)


![Screen Shot 2021-06-11 at 2 45 26 PM](https://user-images.githubusercontent.com/44268796/121735132-b4760900-cac3-11eb-9a93-1e0d31705814.png)


Now, the architecture looks like this:

![Screen Shot 2021-06-11 at 2 44 05 PM](https://user-images.githubusercontent.com/44268796/121734990-8264a700-cac3-11eb-8a95-d83bb81e2a61.png)
  
  























