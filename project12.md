
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

























