# getting started with playbooks

    YAML does not allow the use of tabs
    Must be space between the element parts
    YAML is CASE sensitive
    Indentation is important
    End your YAML file with the .yaml or .yml extension
    YAML is a superset of JSON
    Ansible playbooks are YAML files


## first playbook for installing apache
create a file with your text editor of choice, for example:
```
anthony@ansible-control-host:~/ansible/scripts$ nano install_apache.yml

```
that looks like this
```
---

- hosts: all # we want to run this on all hosts in inventory
  become: true # we want to be able to run the tasks as sudo
  tasks:

  - name: install apache2 package # descriptive name for the task
    apt: # the module that we are going to use 
      name: apache2 # the name of the package 
```

## running the playbook 
`ansible-playbook` command
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml 
BECOME password: 

PLAY [all] **********************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [install apache2 package] **************************************************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.84]

PLAY RECAP **********************************************************************************************************************************************************************
10.0.0.106                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.121                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.84                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
- before running any tasks, ansible will gather information on the hosts
- changed 1 means it changed something in the host
- unreachable 1 if the host was down or there's a networking issue
- failed 1 if well, yknow...
- skipped 1 if the task doesn't meet the condition to be executed
- rescued 1 if we run a task as a rescue if the previous one fails

## running the same playbook to see output difference
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml 
BECOME password: 

PLAY [all] **********************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]

TASK [install apache2 package] **************************************************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]

PLAY RECAP **********************************************************************************************************************************************************************
10.0.0.106                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.121                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.84                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
it didn't have anything to change this time

## modifying the playbook, making sure everytime we run it we install the latest versions available
notice `state: latest`
```
---

- hosts: all # we want to run this on all hosts in inventory
  become: true # we want to be able to run the tasks as sudo
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package # this is a name for the task and it will show up during execution
    apt: # the module that we are going to use
      name: apache2 # the name of the package
      state: latest 

  - name: add php support for apache
    apt:
      name: libapache2-mod-php
      state: latest

```
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml 
BECOME password: 

PLAY [all] **********************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.121]

TASK [update repository index] **************************************************************************************************************************************************
changed: [10.0.0.84]
changed: [10.0.0.121]
changed: [10.0.0.106]

TASK [install apache2 package] **************************************************************************************************************************************************
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.84]

TASK [add php support for apache] ***********************************************************************************************************************************************
changed: [10.0.0.84]
changed: [10.0.0.106]
changed: [10.0.0.121]

PLAY RECAP **********************************************************************************************************************************************************************
10.0.0.106                 : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.121                 : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## playbook to remove apache
 notice `state: absent`
```
---

- hosts: all # we want to run this on all hosts in inventory
  become: true # we want to be able to run the tasks as sudo
  tasks:

  - name: remove apache2 package # descriptive name for the task and it will show up during execution
    apt: # the module that we are going to use
      name: apache2 # the name of the package
      state: absent # the absent state is how we remove packages

  - name: remove php support mod for apache
    apt:
      name: libapache2-mod-php
      state: absent
```

```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass remove_apache.yml 
BECOME password: 

PLAY [all] **********************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [remove apache2 package] ***************************************************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.84]

TASK [remove php support mod for apache] ****************************************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.84]

PLAY RECAP **********************************************************************************************************************************************************************
10.0.0.106                 : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.121                 : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.84                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
## installing apache again
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass install_apache.yml 
BECOME password: 

PLAY [all] **********************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [update repository index] **************************************************************************************************************************************************
changed: [10.0.0.84]
changed: [10.0.0.121]
changed: [10.0.0.106]

TASK [install apache2 package] **************************************************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.84]

TASK [add php support for apache] ***********************************************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.84]

PLAY RECAP **********************************************************************************************************************************************************************
10.0.0.106                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.121                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
