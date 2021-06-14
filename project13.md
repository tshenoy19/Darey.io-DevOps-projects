
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

