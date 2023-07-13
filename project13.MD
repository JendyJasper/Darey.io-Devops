# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
```
import = Static
include = Dynamic
```

- In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.
`git checkout -b dynamic-assignments`
- Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. 
`mkdir dynamic-assignments` and `touch dynamic-assignments/env-vars.yml`
- We will instruct site.yml to include this playbook later.
- The structure looks like this:

<img width="295" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/780ff4cd-e099-4795-8b4d-61563f2707af">

> Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

> For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

- The new layout looks like this:

<img width="295" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/56b59825-447a-4505-af93-d4a38b67c96d">

- paste the instruction below into the env-vars.yml file.
```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

## UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS

- site.yml should now look like this:

```
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: uat-webservers
- name: uat-webservers assignment
  import_playbook: ../static-assignments/uat-webservers.yml
```

## Community Roles
> Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

### Download Mysql Ansible Role

- Browse available roles on https://galaxy.ansible.com/home. We will be using a MySQL role developed by geerlingguy.
- Inside roles directory create your new MySQL role with `sudo ansible-galaxy install geerlingguy.mysql` and rename the folder to MySQL:
`sudo mv geerlingguy.mysql/ mysql`

- Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.






