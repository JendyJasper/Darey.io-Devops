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

- Run the ansible playbook command to test if it works

___________________________________________

<img width="1364" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5fcd6858-cd26-4235-90eb-530e71c2a699">

_____________________________________________

<img width="1364" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/c0c3cb7b-ae1d-43e5-a55b-fd8dc6eede57">


____________________________________________

<img width="522" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/e0fce757-1177-44b3-8c90-100db6caccdd">


# LOAD BALANCER ROLES

- We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:
  - Nginx
  - Apache

- Get ansible roles from ansible galaxy to set up load balancer
- Update both `static-assignment` and `site.yml` files to refer the roles

> Important Hints:
  - Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.
  - Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.
  - Set both values to false like this `enable_nginx_lb: false` and `enable_apache_lb: false`
  - Declare another variable in both roles `load_balancer_is_required` and set its value to `false` as well
  - Update both assignment and site.yml files respectively:

`uat-lb.yml file`

```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

`site.yml file:`
```
- hosts: lb
- name: Loadbalancers assignment
  import_playbook: ../static-assignments/uat-lb.yml
  when: load_balancer_is_required 
```
- Now you can make use of `env-vars\uat.yml` file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.
- You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.
```
enable_nginx_lb: true
load_balancer_is_required: true
```

- The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

- Now when the nginx server is set to run, it skips all the plays on apache server and vice versa. We are using the same port address, port 80, so remember to stop the service of either nginx when you want to run apache load balancer and vice versa.

________________________________________________

<img width="776" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/25a0470e-f319-4742-945a-e0b49bc05235">

Notice how it skips all apache2 plays, this is because nginx load balancer is activated.
_______________________________________________

<img width="776" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/59c009c3-705a-41b4-81f2-17d86ac77b9f">


Notice how it skips all nginx plays, this is because apache2 load balancer is activated.
_______________________________________________

In both scenarios, the web server was reachable on same port because i always stopped the service first on the play before i switched to a new service

<img width="1438" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/71f9e85c-c5df-4ecd-9537-6d8d066d11a2">
