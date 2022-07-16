# managing files
create a new directory named "files" 
```
anthony@ansible-control-host:~/ansible/scripts$ mkdir files
```
create a simple html file within the "files" directory 
```
anthony@ansible-control-host:~/ansible/scripts$ nano files/default_site.html
```
## copy default_site.html to our webservers
we are adding a new task to the web_servers group using the module `copy:`

- site.yml
```
(...)
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

  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy: # module for copyng files
      src: default_site.html # source file
      dest: /var/www/html/index.html  # destination file
      owner: root # user that owns the file
      group: root
      mode: 0644 # set file permissions 
(...)
```
running the new task
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --tags apache --ask-become-pass site.yml
BECOME password: 

PLAY [all] ********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]
ok: [10.0.0.84]
ok: [10.0.0.106]

TASK [install updates (Ubuntu)] ***********************************************************************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.106]
ok: [10.0.0.121]
changed: [10.0.0.84]

TASK [install updates (Rocky)] ************************************************************************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
ok: [10.0.0.213]

PLAY [web_servers] ************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.213]

TASK [install apache2 and php (Ubuntu)] ***************************************************************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.121]

TASK [install apache (httpd) and php (Rocky)] *********************************************************************************************************************************************************************
skipping: [10.0.0.121]
ok: [10.0.0.213]

TASK [copy default html file for site] ****************************************************************************************************************************************************************************
changed: [10.0.0.121]
changed: [10.0.0.213]

PLAY [db_servers] *************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.106]

PLAY [file_servers] ***********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ********************************************************************************************************************************************************************************************************
10.0.0.106                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.121                 : ok=5    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=5    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.84                  : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
## provisioning our workstations
we can not only automate tasks for our servers but our workstations as well using ansible

- adding "workstations" group to our inventory with the IP of our ansible-control-host

inventory 
```
[web_servers]
10.0.0.121
10.0.0.213

[db_servers]
10.0.0.106

[file_servers]
10.0.0.84

[workstations]
10.0.0.76
```
- creating new plays (a.k.a. tasks ) to install the 'unzip' package, and then use the `unarchive` module, which depends on having unzip installed, to copy a binary from the web to our workstations

site.yml
```
(...)
- hosts: workstations
  become: true
  tasks:

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
(...)
```
- copy the ssh ansible key to our workstations
```
anthony@ansible-control-host:~/ansible/scripts$ ssh-copy-id -i ~/.ssh/ansible.pub 10.0.0.76
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/anthony/.ssh/ansible.pub"
The authenticity of host '10.0.0.76 (10.0.0.76)' can't be established.
ED25519 key fingerprint is SHA256:
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
anthony@10.0.0.76's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '10.0.0.76'"
and check to make sure that only the key(s) you wanted were added.
```
- run play
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook --tags workstations --ask-become-pass site.yml
BECOME password: 

PLAY [all] ********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.121]
ok: [10.0.0.106]
ok: [10.0.0.84]
ok: [10.0.0.213]
ok: [10.0.0.76]

TASK [install updates (Ubuntu)] ***********************************************************************************************************************************************************************************
skipping: [10.0.0.213]
ok: [10.0.0.106]
ok: [10.0.0.121]
ok: [10.0.0.84]
changed: [10.0.0.76]

TASK [install updates (Rocky)] ************************************************************************************************************************************************************************************
skipping: [10.0.0.121]
skipping: [10.0.0.106]
skipping: [10.0.0.84]
skipping: [10.0.0.76]
ok: [10.0.0.213]

PLAY [workstations] ***********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.76]

TASK [install unzip] **********************************************************************************************************************************************************************************************
changed: [10.0.0.76]

TASK [install terraform] ******************************************************************************************************************************************************************************************
changed: [10.0.0.76]

PLAY [web_servers] ************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.213]
ok: [10.0.0.121]

PLAY [db_servers] *************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.106]

PLAY [file_servers] ***********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [10.0.0.84]

PLAY RECAP ********************************************************************************************************************************************************************************************************
10.0.0.106                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.121                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.213                 : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.76                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.84                  : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
