
### Ansible Dynamic Assignments (Include) and Community Roles

In this project, the UAT servers will be configured to learn about and practice using new Ansible concepts and modules.
This project will introduce dynamic assignments by using the include module.
The static assignments from Project 12 use the import Ansible module. The module that enables dynamic assignments is include.

```
import = Static
include = Dynamic
```

When the import module is used, all statements are pre-processed at the time playbooks are parsed. When site.yml playbook is executed, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. After the statements are parsed, any changes to the statements encountered during execution will be used.

Note: For most projects, static assignments are preferred as they are more reliable. Dynamic assignments can be challenging in the debugging process. However, they can be useful for environment specific variables as in this project.

#### Introducing Dynamic Assignment Into the structure

In the ```https://github.com/<your-name>/ansible-config-mgt GitHub repository ```, start a new branch and call it dynamic-assignments.
  
![Screen Shot 2021-06-14 at 9 58 45 AM](https://user-images.githubusercontent.com/44268796/121904430-2bdfae80-ccf7-11eb-9a99-2de7ff17268a.png)


Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. site.yml will be instructed to include this playbook later. 

GitHub will have the following structure at this point:
```
├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
```

Since Ansible will be used to configure multiple environments and each of these environments will have certain unique attributes, such as servername, ip-address etc., there has to be a way to set values to variables per specific environment.

Create a folder to keep each environment’s variables file and name it as env-vars. Then for each environment, create new YAML files which will be used to set variables.

The layout should now look like this:
```
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml
```

![Screen Shot 2021-06-14 at 10 11 26 AM](https://user-images.githubusercontent.com/44268796/121906214-e3c18b80-ccf8-11eb-9c3a-1873f196b1ef.png)


Paste the instruction below into the env-vars.yml file:
```yml
---
- name: collate variables from env specific file, if it exists
  include_vars: "{{ item }}"
  with_first_found:
    - "{{ playbook_dir }}/../env_vars/{{ "{{ inventory_file }}.yml"
    - "{{ playbook_dir }}/../env_vars/default.yml"
  tags:
    - always
```

Three things to note from the above code:

The 'include_vars' syntax has been used instead of include. This is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:

include_role
include_tasks
include_vars

In the same version, variants of import were also introduced, such as:

import_role
import_tasks

Special variables {{ playbook_dir }} and {{ inventory_file }} are used in the above code. {{ playbook_dir }} will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. {{ inventory_file }} on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml, so that it picks up the required file within the env-vars folder.

The variables are included using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good practice so that the default values can be set in case an environment specific env file does not exist.


#### Update site.yml with dynamic assignments

Update site.yml file to make use of the dynamic assignment. (At this point, testing cannot be done yet. This is just setting the stage for what is to come).

site.yml should now look like this:
```yml
---
- name: Include dynamic variables 
  hosts: all
  tasks:
    - import_playbook: ../static-assignments/common.yml 
    - include_playbook: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- name: Webserver assignment
  hosts: webservers
    - import_playbook: ../static-assignments/webservers.yml
```

![Screen Shot 2021-06-14 at 11 19 27 AM](https://user-images.githubusercontent.com/44268796/121916714-669b1400-cd02-11eb-9157-b0447d9e95f8.png)

#### Community Roles

It is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. There are tons of roles that have already been developed by other open source engineers. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role. 

#### Download Mysql Ansible Role

Browse the available community roles [here](https://galaxy.ansible.com/home)

This project will make use of a MySQL role developed by [geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql).

On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run:
```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```


![Screen Shot 2021-06-14 at 11 37 50 AM](https://user-images.githubusercontent.com/44268796/121919367-f5109500-cd04-11eb-9e3d-6c260876bd6c.png)

Inside roles directory, create a new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql:
```
ansible-galaxy install geerlingguy.mysql
mv geerlingguy.mysql/ mysql
```

![Screen Shot 2021-06-14 at 11 52 53 AM](https://user-images.githubusercontent.com/44268796/121921500-0f4b7280-cd07-11eb-9efb-b1ff8085288c.png)

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website. The main.yml file in the defaults folder of roles directory is edited as follows:


![Screen Shot 2021-06-14 at 12 03 55 PM](https://user-images.githubusercontent.com/44268796/121923280-c3013200-cd08-11eb-96b6-df413f3805ff.png)



Now it is time to upload the changes into GitHub:


































