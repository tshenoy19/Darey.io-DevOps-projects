
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

























