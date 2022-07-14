# target specific nodes by group
first lets check our files starting point

install_apache.yml
```
---

- hosts: all
  become: true
  tasks:

  - name: update repo index and install apache2 and php (apt)
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
      update_cache: yes
    when: ansible_distribution in ["Ubuntu"," Debian"]

  - name: update repo index and install apache package (httpd) and php (dnf)
    dnf:
      name:
        - httpd
        - php
      state: latest
      update_cache: yes
    when: ansible_distribution == "Rocky"
```
inventory
```
10.0.0.121
10.0.0.106
10.0.0.84
10.0.0.213
```
## creating groups in the inventory file
note: a server can be part of multiple groups, even though there isnt such case in our example

inventory
```
[web_servers]
10.0.0.121
10.0.0.213

[db_servers]
10.0.0.106

[file_servers]
10.0.0.84
```

## creating a new playbook

site.yml
```
---

- hosts: all
  become: true
  pre_tasks: # pre_tasks mandates that this section gets executed first, although it has no effect in our script since it is at the beginning of it

  - name: install updates (Ubuntu)
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install updates (Rocky)
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "Rocky"

- hosts: web_servers
  become: true
  tasks:

  - name: install apache2 and php (Ubuntu)
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache (httpd) and php (Rocky)
    dnf:
      name:
        - httpd
        - php
      state: latest
    when: ansible_distribution == "Rocky"


- hosts: db_servers
  become: true
  tasks:

  - name: install mariadb (mariadb-server) (Ubuntu)
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install mariadb (Rocky)
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "Rocky"


- hosts: file_servers
  become: true
  tasks:

  - name: install samba  (Ubuntu or Rocky)
    package:
      name: samba # this package has the same name in both distributions so it can all be done in one task
      state: latest

```
setting our servers by group
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --ask-become-pass site.yml
BECOME password: 

PLAY [all] ***********************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.121]

TASK [install updates (Ubuntu)] **************************************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.84]
changed: [10.0.0.121]
changed: [10.0.0.106]

TASK [install updates (Rocky)] ***************************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY [web_servers] ***************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ******************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] ************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] *********************************************************************************************************************
changed: [10.0.0.106]

TASK [install mariadb (Rocky)] ***************************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] **************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ******************************************************************************************************************************
changed: [10.0.0.84]

PLAY RECAP ***********************************************************************************************************************************************************
10.0.0.106                 : ok=4    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=4    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```