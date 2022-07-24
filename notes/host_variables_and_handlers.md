# host variables and handlers

## create a file to contain the variables
```
anthony@ansible-control-host:~/ansible/scripts$ mkdir host_vars
```
## create a host variables file for each of our hosts
the name of the file should be the ip address or the hostname or dns name of your server, followed by .yml

- 10.0.0.120.yml
```
anthony@ansible-control-host:~/ansible/scripts/host_vars$ nano 10.0.0.120.yml
```
```
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
```
- 10.0.0.213.yml
```
apache_package_name: httpd
apache_service: httpd
php_package_name: php
```
- 10.0.0.105.yml
```
mariadb_package_name: mariadb-server
mariadb_service: mariadb
```
- 10.0.0.83.yml
```
samba_package_name: samba
samba_service: samba
```

## change your taskbooks to take advantage of host variables
- web_servers/tasks/main.yml
```
- name: install apache and php
  tags: apache,php,httpd
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache service
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

(...)
```
- db_servers/tasks/main.yml
```
- name: install mariadb 
  tags: db,mariadb 
  package:
    name: mariadb-server
    state: latest
```
- file_servers/tasks/main.yml
```
- name: install samba 
  tags: file,samba
  package:
    name: "{{ samba_package_name }}"
    state: latest
``` 



## creating a handler, use `notify: ` to call the handler

- create the handler
```
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ mkdir handlers
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers$ cd handlers/
anthony@ansible-control-host:~/ansible/scripts/roles/web_servers/handlers$ nano main.yml
```
- .../web_servers/handlers/main.yml
```
- name: restart_apache # same name the notify will call
  service:
    name: "{{ apache_service }}"
    state: restarted
```

- web_servers/tasks/main.yml
```
- name: install apache and php
  tags: apache,php,httpd
  package:
    name:
      - "{{ apache_package_name }}"
      - "{{ php_package_name }}"
    state: latest

- name: start and enable apache service
  tags: apache,httpd
  service:
    name: "{{ apache_service }}"
    state: started
    enabled: yes

- name: change email address for admin (Rocky)
  tags: apache,httpd,rocky
  lineinfile: # module to modify a line in a file
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin' # regular expression to find the line to be modified
    line: ServerAdmin anthony@server4 # text of the new line
  when: ansible_distribution == "Rocky"
  notify: restart_apache # notify this handler if something changes

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy: # module for copyng files
    src: default_site.html # source file
    dest: /var/www/html/index.html  # destination file
    owner: root # user that owns the file
    group: root
    mode: 0644 # set file permissions

```
changed the email address to anthony@server4 so we can see a change in the play and it runs the handler
```
anthony@ansible-control-host:~/ansible/scripts$ ansible-playbook site.yml 

PLAY [all] *******************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.105]
ok: [10.0.0.83]
ok: [10.0.0.120]
ok: [10.0.0.213]
ok: [10.0.0.75]

TASK [update repo cache (Ubuntu)] ********************************************************************************************************************
skipping: [10.0.0.213]
changed: [10.0.0.83]
changed: [10.0.0.120]
changed: [10.0.0.105]
changed: [10.0.0.75]

TASK [update repo cache (Rocky)] *********************************************************************************************************************
skipping: [10.0.0.120]
skipping: [10.0.0.105]
skipping: [10.0.0.83]
skipping: [10.0.0.75]
ok: [10.0.0.213]

PLAY [all] *******************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.105]
ok: [10.0.0.83]
ok: [10.0.0.213]
ok: [10.0.0.75]

TASK [base : add ssh key for jarvis] *****************************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.83]
ok: [10.0.0.105]
ok: [10.0.0.213]
ok: [10.0.0.75]

PLAY [workstations] **********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.75]

TASK [workstations : install unzip] ******************************************************************************************************************
ok: [10.0.0.75]

TASK [workstations : install terraform] **************************************************************************************************************
ok: [10.0.0.75]

PLAY [web_servers] ***********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : install apache and php] **********************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : start and enable apache service] *************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

TASK [web_servers : change email address for admin (Rocky)] ******************************************************************************************
skipping: [10.0.0.120]
changed: [10.0.0.213]

TASK [web_servers : copy default html file for site] *************************************************************************************************
ok: [10.0.0.120]
ok: [10.0.0.213]

RUNNING HANDLER [web_servers : restart_apache] *******************************************************************************************************
changed: [10.0.0.213]

PLAY [db_servers] ************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.105]

TASK [db_servers : install mariadb (mariadb-server) (Ubuntu)] ****************************************************************************************
ok: [10.0.0.105]

TASK [db_servers : install mariadb (Rocky)] **********************************************************************************************************
skipping: [10.0.0.105]

PLAY [file_servers] **********************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.0.0.83]

TASK [file_servers : install samba  (Ubuntu or Rocky)] ***********************************************************************************************
ok: [10.0.0.83]

PLAY RECAP *******************************************************************************************************************************************
10.0.0.105                 : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.120                 : ok=8    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
10.0.0.213                 : ok=10   changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.75                  : ok=7    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
10.0.0.83                  : ok=6    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
