# targeting specific tasks, groups, distros, etc. using tags

## tags:
```
---

- hosts: all
  become: true
  pre_tasks: # pre_tasks mandates that this section gets executed first, although it has no effect in our script since it is at the beginning of it

  - name: install updates (Ubuntu)
    tags: always # this will always be executed
    apt:
      upgrade: dist
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install updates (Rocky)
    tags: always
    dnf:
      update_only: yes
      update_cache: yes
    when: ansible_distribution == "Rocky"

- hosts: web_servers
  become: true
  tasks:

  - name: install apache2 and php (Ubuntu)
    tags: apache,apache2,php,ubuntu # different tags, comma separated with no spaces
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache (httpd) and php (Rocky)
    tags: apache,httpd,php,rocky
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
    tags: db,mariadb,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install mariadb (Rocky)
    tags: db,mariadb,rocky
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "Rocky"


- hosts: file_servers
  become: true
  tasks:

  - name: install samba  (Ubuntu or Rocky)
    tags: samba,ubuntu,rocky
    package:
      name: samba # this package has the same name in both distributions so it can all be done in one task
      state: latest
```

## list available tags in a playbook
`ansible-playbook --list-tags`
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --list-tags site.yml

playbook: site.yml

  play #1 (all): all	TAGS: []
      TASK TAGS: [always]

  play #2 (web_servers): web_servers	TAGS: []
      TASK TAGS: [apache, apache2, httpd, php, rocky, ubuntu]

  play #3 (db_servers): db_servers	TAGS: []
      TASK TAGS: [db, mariadb, rocky, ubuntu]

  play #4 (file_servers): file_servers	TAGS: []
      TASK TAGS: [rocky, samba, ubuntu]
```
## running tasks by tag
`ansible-playbook --tags tag --ask-become-pass playbook.yml`
- rocky
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --tags rocky --ask-become-pass site.yml
BECOME password: 

PLAY [all] ****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.213]

TASK [install updates (Ubuntu)] *******************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [install updates (Rocky)] ********************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY [web_servers] ********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache (httpd) and php (Rocky)] *****************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] ********************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] *******************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ***********************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ****************************************************************************************************************************
10.0.0.106                 : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
- db
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --tags db --ask-become-pass site.yml
BECOME password: 

PLAY [all] ****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install updates (Ubuntu)] *******************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [install updates (Rocky)] ********************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY [web_servers] ********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] **************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] ********************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] *******************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ****************************************************************************************************************************
10.0.0.106                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.213                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
- samba
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --tags samba --ask-become-pass site.yml
BECOME password: 

PLAY [all] ****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install updates (Ubuntu)] *******************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [install updates (Rocky)] ********************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY [web_servers] ********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.106]

PLAY [file_servers] *******************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]

TASK [install samba  (Ubuntu or Rocky)] ***********************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ****************************************************************************************************************************
10.0.0.106                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.121                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.213                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
## selecting multiple tags
`ansible-playbook --tags "tag1,tag2" --ask-become-pass playbook.yml`
- apache,db
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --tags "apache,db" --ask-become-pass site.yml
BECOME password: 

PLAY [all] ****************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]

TASK [install updates (Ubuntu)] *******************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]

TASK [install updates (Rocky)] ********************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY [web_servers] ********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ***********************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] *****************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *********************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (mariadb-server) (Ubuntu)] **************************************************************************************
ok: [10.0.0.106]

TASK [install mariadb (Rocky)] ********************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] *******************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ****************************************************************************************************************************
10.0.0.106                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.84                  : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```