
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



































