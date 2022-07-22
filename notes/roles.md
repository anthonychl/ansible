# roles

## changing our site.yml playbook to use roles

```
---

- hosts: all
  become: true
  pre_tasks: # pre_tasks mandates that this section gets executed first, although it has no effect in our script since it is at the beginning of it

  - name: update repo cache (Ubuntu)
    tags: always # this will always be executed
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: update repo cache (Rocky)
    tags: always
    dnf:
      update_cache: yes
    when: ansible_distribution == "Rocky"

- hosts: all
  become: true
  roles:
    - base # the script looks for a folder named base to find tasks

- hosts: workstations
  become: true
  roles:
    - workstations

- hosts: web_servers
  become: true
  roles:
    - web_servers

- hosts: db_servers
  become: true
  roles:
    - db_servers

- hosts: file_servers
  become: true
  roles:
    - file_servers

```
## create a roles directory
```
anthony@ansible-control-host:~/ansible/scripts$ mkdir roles
```
## create a directory, within the roles directory, for each role you want to add
```
anthony@ansible-control-host:~/ansible/scripts$ cd roles

anthony@ansible-control-host:~/ansible/scripts/roles$ mkdir base

anthony@ansible-control-host:~/ansible/scripts/roles$ mkdir workstations

anthony@ansible-control-host:~/ansible/scripts/roles$ mkdir web_servers

anthony@ansible-control-host:~/ansible/scripts/roles$ mkdir db_servers

anthony@ansible-control-host:~/ansible/scripts/roles$ mkdir file_servers
```
## within each role directory, create a tasks directory
```
anthony@ansible-control-host:~/ansible/scripts/roles$ cd base
anthony@ansible-control-host:~/ansible/scripts/roles/base$ mkdir tasks
anthony@ansible-control-host:~/ansible/scripts/roles/base$ cd ..

anthony@ansible-control-host:~/ansible/scripts/roles$ cd workstations
anthony@ansible-control-host:~/ansible/scripts/roles/workstations$ mkdir tasks
anthony@ansible-control-host:~/ansible/scripts/roles/workstations$ cd ..

anthony@ansible-control-host:~/ansible/scripts/roles$ cd web_servers
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ mkdir tasks
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ cd ..

anthony@ansible-control-host:~/ansible/scripts/roles$ cd db_servers
anthony@ansible-control-host:~/ansible/scripts/roles/db_servers$ mkdir tasks
anthony@ansible-control-host:~/ansible/scripts/roles/db_servers$ cd ..

anthony@ansible-control-host:~/ansible/scripts/roles$ cd file_servers
anthony@ansible-control-host:~/ansible/scripts/roles/file_servers$ mkdir tasks
anthony@ansible-control-host:~/ansible/scripts/roles/file_servers$ cd ..
```
## in each tasks directory, create a main.yml file, these are called taskbooks
- base role (the script within this file is actually a place holder, the bootstraping playbook already sets up the ssh key, but later on we can remove or modify the key with this role)
```
anthony@ansible-control-host:~/ansible/scripts/roles$ cd base/tasks/
anthony@ansible-control-host:~/ansible/scripts/roles/base/tasks$ nano main.yml
```
```
- name: add ssh key for jarvis
  tags: always
  authorized_key:
    user: jarvis
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBRsPEBBSXDFUrCKgcQQOqUUB8jasasasasasasas ansible" # copy from ~/.ssh/ansible.pub
```
- workstations
```
anthony@ansible-control-host:~/ansible/scripts/roles/workstations/tasks$ nano main.yml
```
```
- name: install unzip
  tags: unzip,workstations
  package:
    name: unzip

- name: install terraform
  tags: terraform,workstations
  unarchive:
    src: https://releases.hashicorp.com/terraform/1.2.5/terraform_1.2.5_linux_amd64.zip
    dest: /usr/local/bin
    remote_src: yes # stating that we are copying from the web and not a local file
    mode: 0755
    owner: root
    group: root
```
- web_servers
```
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers/tasks$ nano main.yml
```
```
- name: install apache (httpd) and php (Rocky)
  tags: apache,httpd,php,rocky
  dnf:
    name:
      - httpd
      - php
    state: latest
  when: ansible_distribution == "Rocky"

- name: start httpd (Rocky)
  tags: apache,httpd,rocky
  service: # allows to manage services
    name: httpd # name of the service we want to manage
    state: started # set the state of the service
    enabled: yes # set the service to start at boot
  when: ansible_distribution == "Rocky"

- name: change email address for admin (Rocky)
  tags: apache,httpd,rocky
  lineinfile: # module to modify a line in a file
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin' # regular expression to find the line to be modified
    line: ServerAdmin anthony@localhost # text of the new line
  when: ansible_distribution == "Rocky"
  register: httpd_var # setting a variable to capture the state (if something changed)

- name: restart httpd (Rocky)
  tags: apache,httpd,rocky
  service:
    name: httpd
    state: restarted
  when: httpd_var.changed # run this task if httpd_var changed

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy: # module for copyng files
    src: default_site.html # source file
    dest: /var/www/html/index.html  # destination file
    owner: root # user that owns the file
    group: root
    mode: 0644 # set file permissions 
```
for the web servers we also need to create a files directory within web_servers directory and copy the default_site.html file to it
```
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ mkdir files

anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ cp ../../files/default_site.html ./files/default_site.html

anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ ls
files  tasks
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ ls files
default_site.html
```
- db_servers
```
anthony@ansible-control-host:~/ansible/scripts/roles/db_servers/tasks$ nano main.yml
```
```
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
```
- file_servers
```
anthony@ansible-control-host:~/ansible/scripts/roles/file_servers/tasks$ nano main.yml
```
```
- name: install samba  (Ubuntu or Rocky)
  tags: samba,ubuntu,rocky
  package:
    name: samba # this package has the same name in both distributions so it can all be done in one task
    state: latest
```

## run site.yml
notice how now in front of every task it shows the role name
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook site.yml 

PLAY [all] ************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [update repo cache (Ubuntu)] *************************************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.84]
changed: [10.0.0.121]
changed: [10.0.0.106]
changed: [10.0.0.76]

TASK [update repo cache (Rocky)] **************************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [all] ************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.106]
ok: [10.0.0.76]

TASK [base : add ssh key for jarvis] **********************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.84]
ok: [10.0.0.106]
ok: [10.0.0.213]
ok: [10.0.0.76]

PLAY [workstations] ***************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [10.0.0.76]

TASK [workstations : install unzip] ***********************************************************************************************************************************
ok: [10.0.0.76]

TASK [workstations : install terraform] *******************************************************************************************************************************
ok: [10.0.0.76]

PLAY [web_servers] ****************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [web_servers : install apache2 and php (Ubuntu)] *****************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [web_servers : install apache (httpd) and php (Rocky)] ***********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [web_servers : start httpd (Rocky)] ******************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [web_servers : change email address for admin (Rocky)] ***********************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [web_servers : restart httpd (Rocky)] ****************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.213]

TASK [web_servers : copy default html file for site] ******************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

PLAY [db_servers] *****************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [10.0.0.106]

TASK [db_servers : install mariadb (mariadb-server) (Ubuntu)] *********************************************************************************************************
ok: [10.0.0.106]

TASK [db_servers : install mariadb (Rocky)] ***************************************************************************************************************************
skipping: [10.0.0.106]

PLAY [file_servers] ***************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [10.0.0.84]

TASK [file_servers : install samba  (Ubuntu or Rocky)] ****************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ************************************************************************************************************************************************************
10.0.0.106                 : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.121                 : ok=7    changed=1    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
10.0.0.213                 : ok=9    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
10.0.0.76                  : ok=7    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=6    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```